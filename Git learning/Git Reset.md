git reset <hash_code_of_destination_commit>

-> git reset command has three mode. 

1. --hard : In this mode, it will change the head to the destination commit, index (staging area) to same as index of destination commit, and working directory also to same as destination commit.

2. --soft : head will point to the destination commit, working directory will be same as last commit, staging area will be also same as last commit.

3. --mixed (default) : head will point to destination commit, working directory will be same as last commit, staging area will be same as first commit.