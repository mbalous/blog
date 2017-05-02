# Portable SSL-enabled IMAP & SMTP
#### Posted May 2nd, 01:01 AM

### Intro
Many of the ideas I've been playing around with lately are of the distributed, secure & networked kind. Since distributed asynchronous networks are tricky beasts, I want to leverage the existing network of such nature commonly known as email. This quest has led me to some of the hardest to find and buggiest libraries I have ever come across. Either IMAP & SMTP are tricky as hell to get right or there is some kind of mass-psychosis going on, I never felt like dipping appendages into that tar pit enough to tell. The two solid implementations I have found are in the standard libraries of Java & Python.

### Golang
This time around I was trying to get [an idea](https://github.com/andreas-gone-wild/snackis) up and running in Golang. The usually excellent library support is one of the reasons I went with Golang to begin with, but when it comes to email I might as well have been coding in JavaScript. Writing the whole application in Python is out, big Python code-bases have the structural integrity of Jello from my experience; and Java goes too far the other way. No, Golang it is.

### The epiphany
After tearing most of my hair out trying to get something working within Golang, I started looking at calling external code instead; and gradually it dawned on me that writing two tiny servers in Python, one for SMTP and one for IMAP; and remote controlling them via ```stdin``` / ```stdout``` should be possible. The Python solution ended up taking about the same amount of code as the non-working Golang version, with glue included. And it performs better than the naive one-connection-per-request strategy that was all I could get semi-working in Golang, with room to grow. And as an added bonus I now have in my posession a portable solution for the email problem, which will make my life a lot easier. 

### The code
Included below are the two tiny Python servers and the Golang glue-code used to drive them. The communication protocol is kept as simple as possible by using lines of text where possible and json objects where needed.

#### imap.py

```
import json

from email import message_from_string
from email.header import decode_header
from imaplib import IMAP4_SSL
from sys import argv, stdin, stdout

TAG = '__SNACKIS__'

SERVER, PORT, USER, PASSWORD = argv[1:]

imap = IMAP4_SSL(SERVER, int(PORT))
imap.login(USER, PASSWORD)
imap.create(TAG)
imap.select('INBOX')

while True:
    if stdin.readline().strip() == "quit": break
    _, data = imap.uid('SEARCH', None, '(HEADER Subject "{0}")'.format(TAG))

    for uid in data[0].split():
        try:
            res, raw = imap.uid('FETCH', uid, '(RFC822)')
            if res != 'OK': continue
            msg = message_from_string(raw[0][1].decode('utf-8'))
            subj = decode_header(msg['Subject'])[0][0]
    
            json.dump({
                'Id': subj[len(TAG)+1:],
                'SentBy': decode_header(msg['From'])[0][0],
                'Body': msg.get_payload()
            }, stdout)
            print("")
            
            if imap.uid('COPY', uid, TAG)[0] == 'OK':
                imap.uid('STORE', uid , '+FLAGS', '(\Deleted)')
        except Exception as e:
            json.dump({'Error': e.message}, stdout)
	    print("")

        stdout.flush()
    
    print("")
    stdout.flush()
    imap.expunge()

imap.close()
imap.logout()
```

#### imap.go

```
package imap

import (
	"bufio"
	"encoding/json"
	"fmt"
	"io"
	"os/exec"
	"strconv"
)

type Proc struct {
	Cmd *exec.Cmd
	In io.Writer
	Out *bufio.Reader
}

type Msg struct {
	Id, SentBy, Body string
	Error string
}

func Start(server string, port int, user, password string) (proc *Proc, err error) {
	proc = new(Proc)
	
	proc.Cmd = exec.Command("python3",
		"imap.py",
		server,
		strconv.Itoa(port),
		user,
		password)

	if proc.In, err = proc.Cmd.StdinPipe(); err != nil {
		return nil, fmt.Errorf("Failed to get imap stdin: %v", err)
	}

	var out io.Reader
	
	if out, err = proc.Cmd.StdoutPipe(); err != nil {
		return nil, fmt.Errorf("Failed to get imap stdout: %v", err)
	}

	if err = proc.Cmd.Start(); err != nil {
		return nil, err
	}

	proc.Out = bufio.NewReader(out)
	return proc, nil
}

func (self *Proc) Fetch() (msgs []*Msg, err error) {
	if _, err = self.In.Write([]byte("\n")); err != nil {
		return nil, err
	}

	for true {
		var data []byte
		if data, err = self.Out.ReadBytes('\n'); err != nil {
			return nil, fmt.Errorf("Failed to read result: %v", err)
		}

		if len(data) == 1 {
			break
		}
		
		msg := new(Msg)

		if err = json.Unmarshal(data, msg); err != nil {
			return nil, fmt.Errorf("Failed to unmarshal msg: %v", err)
		}

		msgs = append(msgs, msg)		
	}
	
	return msgs, nil
}

func (self *Proc) Stop() (err error) {
	if _, err = self.In.Write([]byte("quit\n")); err != nil {
		return fmt.Errorf("Failed to send quit command: %v", err)
	}

	if err = self.Cmd.Wait(); err != nil {
		return err
	}
	
	return nil
}
```

#### smtp.py

```
import json

from email.mime.text import MIMEText
from smtplib import SMTP
from sys import argv, stdin, stdout

TAG = '__SNACKIS__'

SERVER, PORT, USER, PASSWORD = argv[1:]

smtp = SMTP(SERVER, int(PORT))
smtp.starttls()
smtp.login(USER, PASSWORD)

while True:
    cmd = json.loads(stdin.readline())

    if cmd == 'quit':
        break

    try:
        msg = MIMEText(cmd['Body'])
        msg['From'] = cmd['SentBy']
        msg['To'] = cmd['SentTo']
        msg['Subject'] = '{0} {1}'.format(TAG, cmd['Id'])
    
        smtp.sendmail(cmd['SentBy'], [cmd['SentTo']], msg.as_string())
        
        print("")
    except Exception as e:
        print(e.message)
    
    stdout.flush()

smtp.quit()
```

#### smtp.go

```
package smtp

import (
	"bufio"
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"os/exec"
	"strconv"
)

type Proc struct {
	Cmd *exec.Cmd
	Out *bufio.Reader
	In *json.Encoder
}

type Msg struct {
	Id, SentBy, SentTo, Body string
}

func Start(server string, port int, user, password string) (proc *Proc, err error) {
	proc = new(Proc)
	
	proc.Cmd = exec.Command("python3",
		"smtp.py",
		server,
		strconv.Itoa(port),
		user,
		password)

	var in io.Writer
	if in, err = proc.Cmd.StdinPipe(); err != nil {
		return nil, fmt.Errorf("Failed to get imap stdin: %v", err)
	}

	proc.In = json.NewEncoder(in)
	var out io.Reader
	
	if out, err = proc.Cmd.StdoutPipe(); err != nil {
		return nil, fmt.Errorf("Failed to get imap stdout: %v", err)
	}

	proc.Out = bufio.NewReader(out)
	
	if err = proc.Cmd.Start(); err != nil {
		return nil, err
	}

	return proc, nil
}

func (self *Proc) Send(msg *Msg) (err error) {
	if err = self.In.Encode(msg); err != nil {
		return fmt.Errorf("Failed to encode json: %v", err)
	}

	var res string
	
	if res, err = self.Out.ReadString('\n'); err != nil {
		return fmt.Errorf("Failed to read result: %v", err)
	}

	if res != "\n" {
		return errors.New(res)
	}
	
	return nil
}

func (self *Proc) Stop() (err error) {
	if err = self.In.Encode("quit"); err != nil {
		return fmt.Errorf("Failed to send quit command: %v", err)
	}

	if err = self.Cmd.Wait(); err != nil {
		return err
	}
	
	return nil
}
```

Be well,
A