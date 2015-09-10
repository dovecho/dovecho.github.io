---
layout: post
title:  "Block Robots"
date: 9/10/2015 14:32:22 AM 
---

## BLOCK ROBOTS of a repository
- Create a new branch, named *NEWBRANCH*
    ```
    git checkout -b NEWBRANCH
    git push -u origin NEWBRANCH
    ```
    
- Change the default branch to *NEWBRANCH*
   See settings of your repository.
  
- Remove the master branch
    ```
    git branch -d master
    git push origin :master
    ```
    
*THIS WILL NOT WORK WITH GITHUB-BASED WEBPAGES USING USERNAME.GITHUB.IO*

## Reference
 - [Block Robots by changing default branch](http://stackoverflow.com/questions/15844905/how-to-stop-google-indexing-my-github-repository)
 - [About Robots.txt](http://www.robotstxt.org/)
 - [Baidu Baike about Robots Agreement](http://baike.baidu.com/view/9274458.htm)
