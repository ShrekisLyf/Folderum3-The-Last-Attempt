# Folderum 3

> **FIXED BUILD:** login safely handles missing values, and Folderum now binds to `0.0.0.0:8080` by default so remote browsers can actually reach the inode catastrophe. — `ls` Is The Bootloader

> **No database. No application files. No bootstrap file. No decoder file.**  
> Run the forum with:
>
> ```bash
> ls|sh
> ```
>
> I wanted to prove you could make a forum without SQL. Then I removed the data files. Then I removed the source files. Then I looked at the remaining bootstrap command and decided *that was still far too much dignity*.

Folderum 3 is a deliberately cursed PHP forum whose **entire installed runtime is directories**.

Not “mostly directories.”

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
README.md       ← you are here; GitHub insists words live in files
folderum3.zip   ← the actual directory-only runtime
```

The ZIP archive itself is obviously a file because transporting a directory tree through GitHub without a container is apparently “normal.” **Inside `folderum3.zip` there are zero regular files.**

This is not production software. It is a hostile demonstration of what happens when `mkdir()` receives venture funding.

---

## The entire startup procedure

Install the dependencies, extract the archive, enter the directory, and commit the incantation:

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

There is no loader file.

There is no configuration file.

There is no source file.

There is no database file.

There is no CSS file.

There is no session file.

There is no reason this should exist.

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

# Install somewhere sensible-ish

For a demo server:

```bash
cd /var/www/html
sudo unzip /path/to/folderum3.zip
sudo chown -R www-data:www-data /var/www/html/folderum3
```

The program directories only need to be readable/traversable. The hidden `.data` tree must be writable because that is where the forum commits its inode-based accounting fraud.

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

For access from another machine, set the listen address before launching:

```bash
FOLDERUM_LISTEN=0.0.0.0:8080 ls|sh
```

Then visit:

```text
http://YOUR-SERVER-IP:8080/
```

Your existing Apache/nginx site on port 80 is not inherently disturbed by a separate process listening on 8080.

Check whether 8080 is already occupied:

```bash
sudo ss -ltnp | grep ':8080'
```

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

means:

1. `ls` converts filesystem metadata into shell source.
2. `sh` executes the directory listing.
3. The listing concatenates the Base32 chunks in RAM.
4. `base32 -d` converts those chunks back into PHP source.
5. `php` executes the source directly from stdin.
6. PHP starts its own tiny HTTP server.
7. The forum stores all runtime state as more directories.
8. Computer science files a restraining order.

No temporary source file is required.

The program exists as PHP only during execution.

At rest, it is a collection of unusually opinionated directory entries.

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

The storage engine is **the fact that ext4 lets me name a directory something regrettable**.

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

The transaction department is currently out drinking.

The query planner is `find`.

The database administration client is `tree`.

The schema migration system is “make another folder and hope.”

---

# Users

Register a user and Folderum creates a directory under:

```text
.data/users/
```

For example:

```text
.data/users/alice/
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

This is a joke project whose authentication backend can be audited with `tree`.

The first registered account becomes admin. Later users become normal users.

Password characters are intentionally restricted so nobody can turn the password field into path traversal fan fiction.

---

# Authentication

A normal authentication system might hash a password and compare the result to a database record.

Folderum asks a more spiritually direct question:

```text
Does password_beer123/ exist?
```

If yes:

```text
welcome back
```

If no:

```text
begone
```

Authentication is a scavenger hunt.

---

# Sessions

Sessions are folders too:

```text
.data/sessions/RANDOM_SESSION_ID/
└── user_BASE64URLUSERNAME/
```

The browser gets the random session ID as a cookie.

The server looks for the corresponding directory.

PHP's normal file-backed session machinery was not invited because it kept bringing files into my directory-only household.

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

Random directory names are primary keys now.

I did not solve relational modeling.

I replaced it with carpentry.

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

The inode table is basically Redis if you are sufficiently irresponsible.

---

# Votes

Alice upvotes a post:

```text
votes/up/alice/
```

Bob downvotes it:

```text
votes/down/bob/
```

Score is:

```text
count(up directories) - count(down directories)
```

The vote table is a folder full of one-byte opinions, except we removed the bytes too.

---

# Bans

Ban Alice:

```bash
mkdir .data/users/alice/banned
```

Unban Alice:

```bash
rmdir .data/users/alice/banned
```

No boolean column.

No moderation record.

There is only ontology.

If `banned/` exists, Alice is banned.

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
rmdir .data/users/alice/role_user
mkdir .data/users/alice/role_admin
```

Manually demote:

```bash
rmdir .data/users/alice/role_admin
mkdir .data/users/alice/role_user
```

RBAC implemented using children's wooden blocks.

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

`UPDATE threads SET locked = 1` has left the building.

---

# Browse the database like a caveman with root access

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

`mv` is UPDATE if you have enough confidence.

---

# What happens to port 80?

Nothing automatically.

Folderum's tiny server defaults to:

```text
0.0.0.0:8080
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

This proof of concept is not designed to be safely Internet-facing.

Use localhost, a lab VM, LAN demo, disposable VPS, or another environment where consequences have been thoughtfully minimized.

---

# Security model

The security model is approximately:

> **please don't.**

This build intentionally includes terrible properties for the joke:

- plaintext passwords in directory names
- a tiny hand-written HTTP server
- no HTTPS termination
- no CSRF protection
- simplistic request parsing
- intentionally bizarre persistence
- deliberately non-production authentication

Do not reuse real credentials.

Do not host valuable data in it.

Do not make this your company's new identity provider because “LDAP has files in it somewhere.”

We eliminated SQL injection by eliminating SQL.

Path traversal immediately applied for the vacant position, so user-controlled path components are restricted.

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

But that is cowardice.

The bootstrap did not become smaller; we just hid it in another file/environment.

`ls|sh` is the fun version because all project-specific knowledge remains inside the directory tree itself.

Five visible characters summon the whole forum from filesystem metadata.

That is sufficiently stupid to be beautiful.

---

# Suggested demo sequence

First, show that there are no files:

```bash
find . -type f
```

Then count them:

```bash
find . -type f | wc -l
```

Result:

```text
0
```

Then show the beginning of the directory listing:

```bash
ls | head
```

Explain:

> Those aren't files. Those are directory names, and they're shell code.

Then run:

```bash
ls|sh
```

Open the forum.

Register the first account.

In another terminal:

```bash
tree .data/users
```

Create a category, thread, reply and vote.

Then:

```bash
tree .data/forum
```

Ban somebody:

```bash
mkdir .data/users/bob/banned
```

Lock a thread:

```bash
mkdir .data/forum/Technology/threads/THREAD_ID/locked
```

Finally:

```bash
find . -type f | wc -l
```

Still:

```text
0
```

Finish with:

> **The homepage is recursive `scandir()` with CSS.**
>
> **Our users table is a directory.**
>
> **Our columns are more directories.**
>
> **Our primary keys are random folder names.**
>
> **Our admin panel is a file manager with delusions of grandeur.**
>
> **We eliminated SQL injection by eliminating SQL. Then we eliminated files. Then we made `ls` the bootloader.**

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

If that succeeds, congratulations: the database cluster has elected a leader.

## `ls|sh` prints weird syntax errors

Run:

```bash
LC_ALL=C ls | head
```

Normal GNU `ls` writing to a pipe emits one entry per line. This package's boot directories are deliberately named and ordered for that behavior.

Also make sure you are **inside the extracted `folderum3` directory**. Do not drop random visible files or directories next to the boot directories; plain `ls` is the bootloader and it will attempt to achieve enlightenment through them too.

---

# Backup

Because the entire runtime is directories, back it up normally from outside the purity bubble:

```bash
tar -czf folderum3-backup.tar.gz folderum3
```

Yes, the backup is a file.

The archive is transportation, not theology.

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

Your inode table can now begin the long process of forgiving you.

---

# Folderum's evolutionary tree

### Folderum 1

> What if categories were folders and SQL simply wasn't invited?

### Folderum 2

> What if literally all forum data were directories?

### Folderum 2: Electric Boogaloo

> What if the PHP source were encoded in directory names too?

### Folderum 3

> **What if `ls` was the bootloader?**

At this point I have not solved data storage.

I have weaponized ext4.

I have not invented a database.

I have convinced the VFS to participate in performance art.

---

# Engineering principles

1. If a boolean can be a directory, it is a directory.
2. If a string can be several directories, congratulations, we invented `TEXT`.
3. If a database can solve it, ignore that option until the problem becomes interesting again.
4. Missing directory = false.
5. Existing directory = true.
6. `mkdir()` is INSERT.
7. `rmdir()` is DELETE.
8. `scandir()` is SELECT.
9. `tree` is phpMyAdmin.
10. Inodes are rows if you disrespect both concepts equally.
11. The filesystem is the ORM.
12. The directory listing is executable source code.
13. `ls` is the bootloader.
14. PHP is now merely a guest in a building made of directory entries.
15. **EVERYTHING. IS. FOLDERS.**

---

# Final note

There is a normal and responsible way to build a forum.

This is valuable because it demonstrates, with unusual clarity, that I am aware of that fact and consciously walked in the opposite direction.

Folderum 3 is a joke, a demo, a systems-programming party trick, and a love letter to Unix semantics written with the emotional stability of a corrupted inode table.

The startup command is still:

```bash
ls|sh
```

Five characters.

No files.

A complete web forum emerges from a directory listing.

**God gave us directory entries and apparently nobody specified what we were allowed to name them.**
