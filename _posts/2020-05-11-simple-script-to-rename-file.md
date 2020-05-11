---
layout: post
title: "Simple code to rename file with python"
description: "Simple code to rename file with python"
comments: true
keywords: "python"
---

I have a folder with a lot of file. I need to change all file that have name include '@1x' to ''. So I write a verry simple script to do it.

##Change file's name with python

```python
import os 
  
# Function to rename multiple files 
def main(): 
  
    for count, filename in enumerate(os.listdir("./")): 
        if "@1x" in filename:
        	newfilename = filename.replace("@1x", "")

        	# rename() function will 
        	# rename all the files 
        	#os.rename(src, dst) 
        	print(newfilename)
        	os.rename("./" + filename, "./" + newfilename)
  
# Driver Code 
if __name__ == '__main__': 
      
    # Calling main() function 
    main() 
```