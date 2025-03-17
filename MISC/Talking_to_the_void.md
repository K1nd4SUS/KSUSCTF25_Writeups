On a unix system, what the users are doing is visible to other users.
The w command is our target.
It will be a prefix of step() on average once every 64 attempts.
A partial solution is to spam shell 1, but the answer would not be understandable.
Fortunately, _memory never sees the original (empty) _state.
So reset() will not intercept seed="".
Therefore, we can travel back in time by sending reset without arguments when we see a good answer with step.
To avoid long runs, we can reset the internal secret after every step, triggering a hard reset with reset.
Note: Due to difficulties in making w work in a container the actual script on the server had a modified run().
The modified function intercepts the first argument w and gives the correct answer.
To account for unintended solution the actual run() is called in all the other cases.
EDIT: The original script had a bug that could compromise the availability of the challenge.
Therefore it had to be patched during the competition (there were surely better fix that retained the unintended solution but I wasn't able to implement them at the time). Sorry about that.
Waiting for sh instead of w provided a blind remote shell, this was because the input parameter for run() was not set.

```Python
from subprocess import Popen, PIPE
from base64 import b64decode
from time import sleep

cmd = 'python server.py'
process = Popen(cmd
, shell=True, text=True
, stdin=PIPE, stdout=PIPE)

def get_line():
	line = process.stdout.readline()
	if not line and process.poll() is not None:
		exit()
	print(line, end='', flush=True)
	return line

def get_lines():
	while True:
		line = get_line()
		if not line:
			sleep(.1)
		if line.startswith("?"):
			break

def send(query):
	print(query)
	query += "\n"
	process.stdin.write(query)
	process.stdin.flush()

def step():
	send("step")
	sleep(.1)
	ans = get_line().strip()
	return ans

def unscramble(secret, message):
	secret = b64decode(secret)
	message = b64decode(message)
	ans = bytes(x ^ y for x, y in zip(secret, message))
	try:
		ans = ans.decode()
	except:
		ans = str(ans)
	return ans

while True:
	get_lines()
	ans = step()
	if ans.startswith("w"):
		break
	get_lines()
	send("reset " + ans)
secret = ans
get_lines()
send("reset")
get_lines()
send("shell 1")
while True:
	line = get_line().rstrip()
	if not line:
		sleep(3)
		continue
	if line.startswith("?"):
		break
	if '.' not in line and ':' not in line:
		print(unscramble(secret, line))
		get_lines()
		break
get_lines()
assert False
```
