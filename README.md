# Folderum 3 — `ls` Is The Bootloader

> **No database. No application files. No bootstrap file. No decoder file.** 
> Run the forum with:
>
> ```bash
> ls|sh
> ```
>
> I wanted to prove you could make a forum without SQL. Then I removed the data files. Then I removed the source files. Then I looked at the remaining bootstrap command and decided *that was still far too much dignity*.

Folderum 3 is a deliberately cursed PHP forum whose **entire installed runtime is directories**.

Its not "oh 90% is directories haha thats funny"

Not “the database is directories but the PHP is normal.”

**Directories.**

The source code is encoded into the *names of directories*. Those directory names are also valid shell statements. When `ls` prints them, it accidentally prints a boot program. Pipe that listing into `sh`, and the boot program reconstructs the PHP into memory and pipes it straight into PHP.

```text
folders
  ↓
 ls
  ↓
shell program
  ↓
 sh
  ↓
Base32 source in RAM
  ↓
base32 -d
  ↓
PHP source in RAM
  ↓
 php
  ↓
HTTP server
  ↓
more folders
```

This repository contains two useful things:

```text
README.md       ← you are here; GitHub insists on explaining this shit to you
folderum3.zip   ← the actual directory-only runtime
```

The ZIP archive itself is obviously a file because "A Directory cant be empty ~ Github 2026" **Inside `folderum3.zip` there are zero regular files.**

This is not production software. It is a hostile demonstration of fuck all to traditional Software.

---

## The entire startup procedure

Install the dependencies, extract the archive, enter the directory, and commit Warcrimes against God, CompSci and Yugoslavia:

```bash
sudo apt update
sudo apt install -y php-cli coreutils unzip
unzip folderum3.zip
cd folderum3
ls|sh
```

That last command is the real launcher.

```bash
ls|sh
```

Five characters.

`ls` is the bootloader.

There is no `index.php`.

There is no `run.sh`.

There is no files

There is no reason this should exist.

And theres no Easter Bunny
---

# Requirements

The boot chain intentionally uses fewer than five external commands:

```text
ls
sh
base32
php
```

On Debian/Ubuntu:

```bash
sudo apt update
sudo apt install -y php-cli coreutils unzip
```

`base32` and `ls` come from GNU coreutils on normal Debian/Ubuntu systems.

Check them:

```bash
php -v
base32 --version
ls --version
```

PHP 8+ is recommended.

---

# Install somewhere.

For a (not recommended) production run

```bash
cd /var/www/html
sudo unzip /path/to/folderum3.zip
sudo chown -R www-data:www-data /var/www/html/folderum3
```

The program directories only need to be readable/traversable. The hidden `.data` tree must be writable because that is where the forum commits crimes against people that glow in the dark.

A simple demo setup:

```bash
sudo find /var/www/html/folderum3 -type d -exec chmod 750 {} \;
sudo chown -R www-data:www-data /var/www/html/folderum3
```

Verify the central theological claim:

```bash
find /var/www/html/folderum3 -type f
```

Expected output:

```text
```

Nothing.

Count them if you distrust me, which is an extremely reasonable response to this project:
Also fuck you, if you have to distrust me

```bash
find /var/www/html/folderum3 -type f | wc -l
```

Expected:

```text
0
```

---

# Run it

Folderum 3 defaults to localhost port 8080.

```bash
cd /var/www/html/folderum3
sudo -u www-data sh -c 'ls|sh'
```

Or while already running as the desired user:

```bash
cd /var/www/html/folderum3
ls|sh
```

You should see:

```text
Folderum 3: ls is the bootloader on http://0.0.0.0:8080
Installed tree contains directories only. Ctrl+C to stop.
```

Open:

```text
http://0.0.0.0:8080/
```

Then visit:

```text
http://YOUR-SERVER-IP:8080/
```

Your existing Apache/nginx site on port 80 is not inherently disturbed by a separate process listening on 8080, so dont piss your pants about it
---

# How can `ls|sh` possibly boot PHP?

Run plain `ls` inside `folderum3`.

You will see directory names resembling:

```text
: A0000;x=$x'FQRHG...'
: A0001;x=$x'MVZGK...'
: A0002;x=$x'...'
...
: Z9999;printf %s "$x"|base32 -d|php
```

Those are not files containing shell code.

**Those are the directory names.**

The names were chosen so that the output of `ls` is itself a shell program.

The beginning of each chunk is:

```sh
:
```

which is the shell no-op command. The `A0000`, `A0001`, etc. text gives the directories a deterministic lexical order. Each line then appends a Base32 source chunk to `$x`:

```sh
x=$x'BASE32CHUNK'
```

Finally the lexically-last directory contains:

```sh
printf %s "$x"|base32 -d|php
```

Therefore:

```bash
ls|sh
```

This lists all and executes all
---

# Why Base32?

Because `/` cannot appear inside a Unix filename.

Classic Base64 uses `/`.

I briefly considered arguing with POSIX, but Base32 was easier.

Base32's alphabet is filename-safe for our purposes:

```text
A-Z 2-7 =
```

This lets every source chunk live directly inside a directory name while the shell line around it remains executable.

So the storage engine is not SQLite.

The storage engine is the fact that everything is a file to UNIX
---

# Why is the data directory called `.data`?

Because default `ls` does not print dot-directories.

The launcher relies on plain:

```bash
ls
```

printing only the boot directories.

Mutable application state therefore lives under:

```text
.data/
```

which remains invisible to the boot listing.

So the directory tree starts roughly like:

```text
folderum3/
├── : A0000;x=$x'...'/
├── : A0001;x=$x'...'/
├── : A0002;x=$x'...'/
├── ...
├── : Z9999;printf %s "$x"|base32 -d|php/
└── .data/
    ├── forum/
    ├── sessions/
    └── users/
```

`ls` sees the executable sculpture.

PHP sees `.data`.

I see no problem here.

---

# The database

There isn't one in the traditional sense.

Folderum's ORM is:

```text
is_dir()
mkdir()
rmdir()
scandir()
```

The query planner is `find`.

The database administration client is `tree`.


---

# Users

Register a user and Folderum creates a directory under:

```text
.data/users/
```

For example:

```text
.data/users/bob/
├── password_beer123/
└── role_admin/
```

or:

```text
.data/users/bob/
├── password_hunter2/
└── role_user/
```

Yes, passwords are deliberately stored in plaintext directory names.

No, this is not remotely safe.

No, you should not use a real password.

The first registered account becomes admin. Later users become normal users.

Password characters are intentionally restricted so nobody can turn the password field into a file injection

---

# Authentication

A normal authentication system might hash a password and compare the result to a database record.

Folderum asks the real questions:

```text
Does password_beer123/ exist?
```

If yes:

```text
welcome back
```

If no:

```text
haha
```
Authentication is a joke to me
---

# Sessions

Sessions are folders too:

```text
.data/sessions/RANDOM_SESSION_ID/
└── user_BASE64URLUSERNAME/
```

The browser gets the random session ID as a cookie.

The server looks for the corresponding directory.

PHP's normal file-backed session machinery is bullshit so i removed the need for one

---

# Categories

A forum category is basically:

```text
.data/forum/Technology/
└── threads/
```

Creating a category is morally equivalent to:

```bash
mkdir -p .data/forum/Technology/threads
```

`INSERT INTO categories` has been deprecated in favor of asking the kernel nicely.

---

# Threads

Threads live below a category:

```text
.data/forum/Technology/threads/0123456789abcdef/
├── author_BASE64URL/
├── posts/
└── title/
    ├── 000000_BASE64URLCHUNK/
    └── ...
```

This is to spit into a hackers face

---

# Posts

Posts are more directories:

```text
posts/POST_ID/
├── author_BASE64URL/
├── body/
│   ├── 000000_BASE64URLCHUNK/
│   ├── 000001_BASE64URLCHUNK/
│   └── ...
└── votes/
    ├── down/
    └── up/
```

Long text is Base64URL-encoded, split into chunks, and represented by ordered directory names.

I implemented a `TEXT` datatype with `mkdir()`.

---

# Votes

Bob upvotes a post:

```text
votes/up/bob/
```

Bob downvotes it:

```text
votes/down/bob/
```

Score is:

```text
count(up directories) - count(down directories)
```

The vote table is a math problem even the most retarded first graders can solve.

---

# Bans

Ban bob:

```bash
mkdir .data/users/bob/banned
```

Unban bob:

```bash
rmdir .data/users/bob/banned
```

No boolean column.

No moderation record.

There is only ontology.

If `banned/` exists, bob is banned.

The admin UI exposes ban/unban actions as well.

The admin panel is a file manager with delusions of grandeur.

---

# Roles

A role is represented by a directory such as:

```text
role_admin/
role_user/
```

Manually promote somebody:

```bash
rmdir .data/users/bob/role_user
mkdir .data/users/bob/role_admin
```

Manually demote:

```bash
rmdir .data/users/bob/role_admin
mkdir .data/users/bob/role_user
```
---

# Thread locking

The reply handler understands a thread-level:

```text
locked/
```

So if a thread lives at:

```text
.data/forum/Technology/threads/0123456789abcdef/
```

lock it with:

```bash
mkdir .data/forum/Technology/threads/0123456789abcdef/locked
```

Unlock it:

```bash
rmdir .data/forum/Technology/threads/0123456789abcdef/locked
```

This feels like a ftp forum tbh
---

# Browse the database like our founding fathers did (tally ho, lads)

Install `tree` if desired:

```bash
sudo apt install -y tree
```

Then:

```bash
tree .data
```

List users:

```bash
find .data/users -mindepth 1 -maxdepth 1 -type d
```

Find admins:

```bash
find .data/users -type d -name role_admin
```

Find banned users:

```bash
find .data/users -type d -name banned
```

Find locked threads:

```bash
find .data/forum -type d -name locked
```

`tree` is the database administration client.

`find` is the query language.

`mkdir` is INSERT.

`rmdir` is DELETE.

`mv` is UPDATE if you have the balls

---

# What happens to port 80?

Nothing automatically.

Folderum's tiny server defaults to:

```text
0.0.0.1:8080
```

Your normal Apache/nginx setup can remain on port 80.

Conceptually:

```text
Apache/nginx       :80
Folderum 3         :8080
```

If you deliberately bind Folderum to all interfaces:

```bash
FOLDERUM_LISTEN=0.0.0.0:8080 ls|sh
```

then port 8080 becomes reachable according to your firewall/network configuration.


---

# Security model

The security model is approximately:
 **yes**

This build intentionally includes terrible properties for the joke:

- plaintext passwords in directory names
- a tiny hand-written HTTP server
- no HTTPS termination
- simplistic request parsing
- intentionally bizarre persistence
- deliberately non-production authentication

Do not reuse real credentials.

Do not host valuable data in it.

We eliminated SQL injection by eliminating SQL.


---

# The five-character bootloader

Here it is again because this is the entire reason Folderum 3 exists:

```bash
ls|sh
```

Character count:

```text
l s | s h
1 2 3 4 5
```

There is no hidden alias.

There is no `folderum` executable installed elsewhere.

There is no `.profile` trick.

There is no shell function.

There is no bootstrap file.

`ls` genuinely prints the program that `sh` executes.

The actual application source is reconstructed only in the pipeline.

This is what happens when “everything is a file” meets somebody who took that as a personal insult.

---

# Why not make the startup one character?

Easy: define an alias.

```bash
alias x='ls|sh'
```

Then run:

```bash
x
```

But that is not why i made this.
 Theres something beautiful in the fact this can exist, who knows - maybe ill make a video game with this tech

---

# Troubleshooting

## `base32: command not found`

Debian/Ubuntu:

```bash
sudo apt install -y coreutils
```

## `php: command not found`

```bash
sudo apt install -y php-cli
```

## Port 8080 is already occupied

```bash
sudo ss -ltnp | grep ':8080'
```

Choose another port:

```bash
FOLDERUM_LISTEN=127.0.0.1:8081 ls|sh
```

## Permission denied while registering/posting

If running as `www-data`:

```bash
sudo chown -R www-data:www-data /var/www/html/folderum3/.data
sudo find /var/www/html/folderum3/.data -type d -exec chmod 750 {} \;
```

Test database connectivity, enterprise edition:

```bash
sudo -u www-data mkdir /var/www/html/folderum3/.data/fuck_yeah_it_writes
sudo -u www-data rmdir /var/www/html/folderum3/.data/fuck_yeah_it_writes
```

If that succeeds: congratulations

## `ls|sh` prints weird syntax errors

Run:

```bash
LC_ALL=C ls | head
```

Normal GNU `ls` writing to a pipe emits one entry per line. This package's boot directories are deliberately named and ordered for that behavior.

Also make sure you are **inside the extracted `folderum3` directory**. Do not drop random visible files or directories next to the boot directories; plain `ls` is the bootloader and it will attempt to achieve its build process with their names as commands.
---

# Backup

Because the entire runtime is directories, back files up normally

```bash
tar -czf folderum3-backup.tar.gz folderum3
```

Yes, the backup is a file - boohoo

---

# Reset data

Stop Folderum first, then remove and recreate the hidden data roots:

```bash
rm -rf .data/users .data/forum .data/sessions
mkdir -p .data/users .data/forum .data/sessions
```

The next registered user becomes admin again.

---

# Uninstall

```bash
rm -rf folderum3
```

---

# Brief history of Folderum

### Folderum 1

> What if no SQL

### Folderum 2

> What if not even txt

### Folderum 2: Electric Boogaloo

> What if the PHP source were encoded in directory names too?

### Folderum 3

> **What if `ls` was the bootloader?**

At this point I have not solved data storage.

I have made what even i dont understand, nor simply dont want to
I can read the hashes without decoding atp

---

---

# Final note

There is a normal and responsible way to build a forum.

But What i describe as Adderal filled mania mixed with absinthe is the way to go

The startup command is still:

```bash
ls|sh
```

Five characters.

No files.

A complete web forum emerges from a directory listing.

**Everything is a file ~ UNIX**
***Everything is a Folder, fuck you UNIX ~ me***
