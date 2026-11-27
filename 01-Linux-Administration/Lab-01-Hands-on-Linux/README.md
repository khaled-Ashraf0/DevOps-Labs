# Lab 01: Hands-on Linux Fundamentals

Hands-on practice on **Rocky Linux 9** covering file and directory operations, hidden files, text manipulation, permissions, and local user/group administration.

---

## 1. Create Files & Directories

```bash
# 1. Create directory 'lab1' and an empty file 'file1.txt' inside it
mkdir lab1 && touch lab1/file1.txt

# 2. Create 5 empty files (file1.txt to file5.txt) inside 'lab2'
mkdir lab2 && touch lab2/file{1..5}.txt

# 3. Create nested directory structure: lab3/parent/child
mkdir -p lab3/parent/child

# 4. Create 'docs' directory and a text file with initial content
mkdir docs && echo "Initial documentation file" > docs/notes.txt

# 5. Create 'project' directory with read-only permissions for others (rwxr-xr--)
mkdir project && chmod 754 project

# 6. Copy a directory structure recursively
cp -r lab3 lab3_backup

```
## 2. Hidden & Non-Hidden Files
```bash
# 1. Create a hidden file in the home directory
touch ~/.hidden1

# 2. Create a directory with both hidden and non-hidden files
mkdir mixed_dir && touch mixed_dir/visible.txt mixed_dir/.secret.txt

# 3. List all files (including hidden) and list hidden files only
ls -la mixed_dir
ls -ld mixed_dir/.?*

# 4. Delete a hidden file and verify removal
rm ~/.hidden1 && ls -la ~

# 5. Create 5 hidden files at once using brace expansion
touch mixed_dir/.hidden{1..5}

# 6. Find all hidden files inside a directory
find mixed_dir -maxdepth 1 -name ".*" -type f

```
## 3. Copy & Rename Operations
```bash
# 1. Copy a file from one directory to another
cp lab1/file1.txt docs/

# 2. Rename a file within the same directory
mv docs/file1.txt docs/renamed_file1.txt

# 3. Copy a directory recursively
cp -r docs docs_copy

# 4. Copy a file and rename it in a single step
cp lab2/file1.txt lab2/file1_archive.txt

```
## 4. Writing & Appending Lines to Files
```bash
# 1. Create a file and write the first line
echo "Line 1: System initialization" > app.log

# 2. Append a second line without overwriting existing content
echo "Line 2: Service check completed" >> app.log

# 3. Edit the file manually using nano
nano app.log

# 4. Write multiple lines to a file using cat and EOF redirection
cat << 'EOF' > multi_line.txt
Server: Rocky Linux 9
Role: DevOps Lab Environment
Status: Active
EOF

```
## 5. Users & Groups Management
```bash
# 1. Create a new user 'user1'
sudo useradd user1

# 2. Create a new group 'group1'
sudo groupadd group1

# 3. Add 'user1' to 'group1'
sudo usermod -aG group1 user1

# 4. Create a user with a custom home directory
sudo useradd -m -d /opt/custom_home user2

# 5. Create a second group and append 'user1' to it
sudo groupadd group2
sudo usermod -aG group2 user1

# 6. List all local users on the system
cut -d: -f1 /etc/passwd

# 7. Remove a user along with their home directory
sudo userdel -r user2

# 8. Delete a group
sudo groupdel group2

```

