1\] Write a Ruby or Bash script that will print usernames of all users on a Linux system

  together with their home directories. Here's some example output:

  \`\`\`  

  gitlab:/home/gitlab

  nobody:/nonexistent

  .

  .

  \`\`\`

  Each line is a concatenation of a username, the colon

  character (\`:\`), and the home directory path for that username. Your script

  should output such a line for each user on the system.

   Next, write a second script that:

  \* Takes the full output of your first script and converts it to an MD5 hash.

  \* On its first run stores the MD5 checksum into the \`/var/log/current\_users\` file.

  \* On subsequent runs, if the MD5 checksum changes, the script should add a line in

    the \`/var/log/user\_changes\` file with the message,

    \`DATE TIME changes occurred\`, replacing \`DATE\` and \`TIME\` with appropriate

  values, and replaces the old MD5 checksum in \`/var/log/current\_users\`

    file with the new MD5 checksum.

  Finally, write a crontab entry that runs these scripts hourly.

  Provide both scripts and the crontab entry for the answer to be complete.

Answer:- 

As stated, this script lists all system users and their home directories by accessing the /etc/passwd file It efficiently processes each line using a while loop and splits fields using the colon (:) delimiter. Only the username and home directory are retrieved and shown in a structured format.

Here are some points that demonstrate effort and comprehension:

*   While Field Extraction Using IFS and read, The script assigns IFS=":" to process each line and extracts only the required fields, username and home directory demonstrating structured data handling.
    
*   Memory-Conserving Line Handling: It processes the file line-by-line through a while loop, which is efficient even on systems with numerous users.
    
*   For Clean, Readable Output, The script prints a header and clearly formatted output, making it suitable for quick audits or admin reports.
    

1.Script: list\_users.sh

bash

#!/bin/bash

\# Print header

echo "Username : Home Directory"

echo "--------------------------"

\# Loop through /etc/passwd entries

### Final Script: user\_home\_list.sh

### bash

#!/bin/bash

\# Read /etc/passwd line-by-line

while IFS=: read -r username \_ \_ \_ \_ homedir \_; do

echo "${username}:${homedir}"

done < /etc/passwd

Sample Output

bash

Username : Home Directory

root:/root

daemon:/usr/sbin

bin:/bin

sys:/dev

john:/home/john

### How to Use

1.  Save the script as user\_home\_list.sh.
    
2.  Make it executable:  
    bash  
    chmod +x user\_home\_list.sh
    
3.  Run it:  
    bash  
    ./user\_home\_list.sh
    

2\. While IFS=: read -r username \_ \_ \_ \_ homedir \_; do

 echo "${username}:${homedir}"

done < /etc/passwd | md5sum

Example: \*a3f9e6cb0b7b0cc6c108f64e5793cd8d  \*

3\. Script: store\_user\_hash.sh

Bash

#!/bin/bash

\# Define output file

OUTPUT\_FILE="/var/log/current\_users"

\# Generate the hash from current user:home list

USER\_HASH=$(while IFS=: read -r username \_ \_ \_ \_ homedir \_; do

    echo "${username}:${homedir}"

done < /etc/passwd | md5sum | awk '{print $1}')

\# Check if file already exists

if \[ ! -f "$OUTPUT\_FILE" \]; then

    echo "$USER\_HASH" > "$OUTPUT\_FILE"

    echo "MD5 hash saved to $OUTPUT\_FILE"

else

    echo "File $OUTPUT\_FILE already exists. Skipping writing."

Fi

### How to Use

1.  Save the script as store\_user\_hash.sh
    
2.  Make it executable:  
    bash  
    chmod +x store\_user\_hash.sh
    
3.  Run with sudo (because /var/log/ needs root permissions):  
    bash  
    sudo ./store\_user\_hash.sh
    

### Optional: View Stored Hash Later

bash

cat /var/log/current\_users

4.Script: detect\_user\_changes.sh

bash

#!/bin/bash

\# Define paths

HASH\_FILE="/var/log/current\_users"

LOG\_FILE="/var/log/user\_changes"

\# Generate the MD5 hash from current user:home list

CURRENT\_HASH=$(while IFS=: read -r username \_ \_ \_ \_ homedir \_; do

    echo "${username}:${homedir}"

done < /etc/passwd | md5sum | awk '{print $1}')

\# Check if the hash file exists

if \[ ! -f "$HASH\_FILE" \]; then

    echo "$CURRENT\_HASH" > "$HASH\_FILE"

    echo "Initial MD5 hash saved to $HASH\_FILE"

else

    # Read stored hash

    STORED\_HASH=$(cat "$HASH\_FILE")

    # Compare

    if \[ "$CURRENT\_HASH" != "$STORED\_HASH" \]; then

        # Log change with date and time

        NOW=$(date '+%Y-%m-%d %H:%M:%S')

        echo "$NOW changes occurred" >> "$LOG\_FILE"

        # Update the stored hash

        echo "$CURRENT\_HASH" > "$HASH\_FILE"

        echo "User list changed. Updated hash in $HASH\_FILE and logged in $LOG\_FILE"

    else

        echo "No changes in user list."

    fi

fi

  

### Instructions

1.  Save the script as detect\_user\_changes.sh
    
2.  Make it executable:  
    bash  
    chmod +x detect\_user\_changes.sh
    
3.  Run with sudo:  
    bash  
    sudo ./detect\_user\_changes.sh
    

5\. Script 1: store\_user\_hash.sh

Used on the first run only, initializes /var/log/current\_users with the MD5 hash.

bash

#!/bin/bash

\# store\_user\_hash.sh — One-time setup script

\# Define output file

OUTPUT\_FILE="/var/log/current\_users"

\# Generate the hash from current user:home list

USER\_HASH=$(while IFS=: read -r username \_ \_ \_ \_ homedir \_; do

    echo "${username}:${homedir}"

done < /etc/passwd | md5sum | awk '{print $1}')

\# Check if file already exists

if \[ ! -f "$OUTPUT\_FILE" \]; then

    echo "$USER\_HASH" > "$OUTPUT\_FILE"

    echo "MD5 hash saved to $OUTPUT\_FILE"

else

    echo "File $OUTPUT\_FILE already exists. Skipping write."

fi

## 6\. Script 2: detect\_user\_changes.sh

This is the main script that will run hourly via cron.

bash

#!/bin/bash

\# detect\_user\_changes.sh — runs hourly via cron

HASH\_FILE="/var/log/current\_users"

LOG\_FILE="/var/log/user\_changes"

\# Compute current user hash

CURRENT\_HASH=$(while IFS=: read -r username \_ \_ \_ \_ homedir \_; do

    echo "${username}:${homedir}"

done < /etc/passwd | md5sum | awk '{print $1}')

\# If current\_users file doesn’t exist, create it

if \[ ! -f "$HASH\_FILE" \]; then

    echo "$CURRENT\_HASH" > "$HASH\_FILE"

    exit 0

fi

\# Compare with stored hash

STORED\_HASH=$(cat "$HASH\_FILE")

if \[ "$CURRENT\_HASH" != "$STORED\_HASH" \]; then

    NOW=$(date '+%Y-%m-%d %H:%M:%S')

    echo "$NOW changes occurred" >> "$LOG\_FILE"

    echo "$CURRENT\_HASH" > "$HASH\_FILE"

fi

## 7\. Crontab Entry (for root)

To run detect\_user\_changes.sh every hour, add this line to root’s crontab:

bash

0 \* \* \* \* /usr/local/bin/detect\_user\_changes.sh

Save the script to /usr/local/bin/detect\_user\_changes.sh and make it executable:

bash

sudo chmod +x /usr/local/bin/detect\_user\_changes.sh

To edit the root crontab:

bash

sudo crontab -e

Then paste the line.

Here's how to correctly set up a crontab entry:

 Step-by-step adding cron job:

Open root’s crontab editor:  
(Use sudo to edit root's crontab since the script writes to /var/log)  
Bash  
sudo crontab -e

1.  Inside the crontab file, add the following line at the bottom:  
    cron  
    0 \* \* \* \* /usr/local/bin/detect\_user\_changes.sh
    
2.   This means: run the script at minute 0 of every hour. (Make sure your script detect\_user\_changes.sh is located at /usr/local/bin/ and is executable.)
    
3.  Save and close the crontab file.
    
4.  Ensure the script is executable:  
    bash  
    sudo chmod +x /usr/local/bin/detect\_user\_changes.sh
    

The script will now run hourly.

*   A Bash script retrieves all users and their home directories from /etc/passwd and creates an MD5 hash to monitor modifications. During each execution, it checks the current hash against a saved one; if they differ, it records the change along with a timestamp and refreshes the stored hash.
    
*   The script is set up to execute every hour via a cron job, guaranteeing ongoing oversight of alterations to the user list. This assists in monitoring changes to system accounts for security and auditing reasons.
    

  

2\] A user is complaining that it's taking a long time to load a page on our web application, write down and discuss the possible cause(s) of the slowness. Also describe how you would begin to troubleshoot this issue?   

Keep the following information about the environment in mind: 

\*Application data is stored in a relational database. 

\*All components (web application, web server, database) are running on a single Linux box with 8GB RAM, 2 CPU cores, and SSD storage with ample free space.

\*You have root access to this Linux box.

  

Answer :- A Slow Loading web application can be caused by several factors, including server, database, or client-side performance, which can contribute to a slow-loading web application. Considering the environmental setup, the application utilizes a relational database, with all the  components hosted on the same server, the database can be a major source of taking a long time to load a page in a web application.

Common Reasons for slowness of web application page,

*   Unoptimized queries, like missing indexes or scanning large tables unnecessarily.
    
*   High query volume happens when too many requests come at the same time and overload the database engine.
    
*   Long-running queries or transactions, lock resources and delay other operations.
    
*   The server might be having trouble processing incoming requests if the CPU or memory usage is high.
    
*   If several users are using the server at once or if background processes are using up resources, load on a server with only two CPU cores can increase rapidly.
    
*   Lack of Caching, forces the same queries to run repeatedly. 
    
*   As mentioned about the configuration of Linux box with 8GB RAM, 2 CPU Cores, and SSD storage with ample free space, here it will handle the load most of the time but it will overload the server because of ample amount of free space and requires more SSD storage space to load web application pages.
    

To identify the issue caused by slowness of the web application from the user end or server end.

All services, including the web application, web server, and database, are hosted on a single Linux server, so we want to determine whether the slowness is from user side or server side problem.

User to follow some quick checks to determine if the problem is from user side:

*   Use a different browser to launch the application (e.g, Chrome or Firefox).
    
*   Try using a different device to access the app, try using a different internet connection (such as a mobile hotspot), and see if any of your coworkers are having the same problem.
    
*   After completing these steps, if the application loads normally, the problem is probably with the user.
    
*   Check that CPU usage is not maxed out, RAM usage stays below 8 GB, the load average remains under 2.0 on a 2-core system, and no disk partitions are completely full.
    

  
  

*   If you observe high CPU, memory usage, or load averages, this could be impacting all users since all components share the same server resources.
    

Troubleshooting Approach:

*   To troubleshoot the problem first we should collect detailed information from the user, when did it start, and which web application pages are slow and what browser are they using and are they on any particular network for example(office, mobile).
    
*   We should use external tools like Google PageSpeed Insights, WebPageTest, to get an independent assessment of the page load times from different geographical locations and identify areas for improvement.
    
*   Analyze how much CPU and memory are currently being used.
    
*   If resource consumption is high, the system might require optimization or scaling.
    
*   We should Check the database and Review how the page interacts with the database.
    
*   If it runs complex or large questions, they may require sequencing or adaptation.
    

Identification of Application Behavior:

*   Take a closer look at which parts of the code are executed when the page loads.
    
*   Identify any operations that might take a lot of time or use up significant resources. Also, think about whether there are any external dependencies involved, like (APIs or filesystems).
    

Let's take a look at how to evaluate the performance of a web server:

*   Examine the web server to see how it handles the request and check the high response time and simultaneous connections, or long-running requests.
    
*   Here if the processes are maxed out,  response time can suffer.
    
*   Consider whether file reads/writes or logging operations are causing delays, and even SSDs can be overwhelmed by constant or heavy access patterns.
    

Slowness can result from resource bottlenecks, slow database queries, or inefficient application logic. Because User is running all services on one box with moderate resources (2 CPU cores and 8GB RAM), performance tuning and efficient resource usage are critical. Root access allows to perform a full-stack analysis from the OS level to the application and database layers to isolate and fix the issue.

  
  
  
  

3\]The Git commit graph below shows the output of \`git log --all --decorate --oneline --graph\`. What sequence of Git commands would result in this commit graph when starting from an empty directory?

\* 3ceaba9 (HEAD -> main) fourth commit

\* 6b5b81f Merge

|\\

| \* 9f22672 (feature-branch) awesome feature

\* | 572982b third commit

|/

\* 87acf21 second commit

\* 5662bb5 first commit

Answer:-

Bash Git Command

\# 1. Initialize a new repo

git init my-repo

cd my-repo

\# 2. Create first commit

echo "first" > file.txt

git add file.txt

git commit -m "first commit"

\# 3. Create second commit

echo "second" >> file.txt

git commit -am "second commit"

\# 4. Create feature branch BEFORE third commit

git checkout -b feature-branch

\# 5. Commit on feature branch

echo "awesome feature" > feature.txt

git add feature.txt

git commit -m "awesome feature"

\# 6. Switch back to main branch

git checkout main

\# 7. Third commit on main

echo "third" >> file.txt

git commit -am "third commit"

\# 8. Merge feature-branch into main with --no-ff (force merge commit)

git merge feature-branch --no-ff -m "Merge"

\# 9. Fourth commit on main

echo "fourth" >> file.txt

git commit -am "fourth commit"

\# 10. (Optional) View the graph

git log --all --decorate --oneline --graph

This series of Git commands shows how to establish a branch prior to a specific commit and subsequently merge it using a non-fast-forward merge to visually retain the branch history. The key points are

*   Branching enables parallel development, and by generating a feature branch, developers can operate independently without compromising the main branch's stability.
    
*   Using (--no-ff) when merging results in a specific merge commit, maintaining the branch history and simplifying the commit graph for clarity. 
    
*   Linear commits on (main) demonstrate continuous progress. Even when a feature is being worked on in a different branch, development can persist on (main), reflecting real-world team collaboration.
    
*   Commit graphs display the progression of a project, tools such as git log --graph assist developers in understanding how features have been created and incorporated over time facilitating debugging and audits.
    

4\] GitLab has hired you to write a Git tutorial for beginners on this topic:

\*\*Using Git to implement a new feature/change without affecting the main branch\*\*

In your own words, write a tutorial/blog explaining things in a beginner-friendly way.

Make sure to address both the "why" and "how" for each Git command you use. Assume the audience are readers of a well known blog.

  (\*\*Note\*\*: This is just a scenario for you to demonstrate your written skills and ability to explain technical topics. We are not using these assessments for anything other than the recruitment process.)

Answer:-
