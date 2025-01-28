
1. **Three Trees in Git**:
    
    - Git manages three main "trees" or states:
        
        - **HEAD**: Points to the latest commit on the current branch.
            
        - **Index (Staging Area)**: Tracks changes that are staged for the next commit.
            
        - **Working Directory**: Represents the current state of files in your project.



-------------------------<<<<<<<<<<<>>>>>>>>>>>>--------------------



-> git restore file_name command will change the file to match the same as of last commit.

ex: suppose in my last commit, md.txt have "Hey, there. I hope you are doing well"

now i have modified it to contain "Hey, there. I hope you are doing well. Nice to meet you"

-> so when i run the command git restore md.txt, the fill will look like "Hey, there. I hope you are doing well"

-> if i stage the changes using git add md.txt. then to unstage it git restore --staged md.txt. this command will not change the md.txt file to "Hey, there. I hope you are doing well". but md.txt file will look same as "Hey, there. I hope you are doing well. Nice to meet you". 


-------------------------<<<<<<<<<<<>>>>>>>>>>>>--------------------

