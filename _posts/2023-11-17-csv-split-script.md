---
layout: post
title:  "A simple script for splitting csv"
categories: [tools]
tags: [windows,python]
---

# A splitting script

Every now and then we need to send a bunch of .csv, which are limited to a certain number of lines. So I devised a simple solution, if I need .csv of 100 lines from a 1000 line original file, I run the script. 

## How

We import os and sys lib, to create files and get parameters respectively. Needs to match the correct number of parameters.

```py
import os
import sys

if len(sys.argv) != 2:
    print("Uso: python divide-csv.py <arquivo>")
else:
```

## Variables

arg1
: receives the argument, which is the file that will be splitted

input_filename
: Path + arg1

output_filename_pattern
: pattern for the output files

line_limit
: the limit number of lines to output in to the new .csv

line, i, file_number, start
: used in the loop

```python
else:
    arg1 = sys.argv[1]
    print(f"Arquivo: {arg1}")
    input_filename = rf"C:\CSV\{arg1}"
    output_filename_pattern = r"venc_aprox_"
    line_limit = 150
    line = 0
    i = 0
    file_number = 0
    start = 0
```

How it work
: It receives the origin file, get the total number of lines, and for each line untill the line limit, it appends the line to a new file

```python
    with open(input_filename, 'r') as input_file:
        lines = input_file.readlines() 
        total_lines = len(lines) 
    
        while line <= total_lines: 
            if  i == line_limit or line == total_lines:
                file_number += 1 
                filename = f"{output_filename_pattern}{file_number}.csv"
                with open(filename, 'w') as output_file: 
                    output_file.write("telefone\n") 
                    output_file.writelines(lines[start:line]) 
                start = line 
                i = 0 
                print(filename)
            
            i += 1 
            line += 1 
```
## Saving the script

Save the name as you wish, I'll call it _divide-csv.py_

## Executing

On the command line, call for:

```powershell
python3 ./divide-csv.py <file-name>
```

## Requirements
1. Python
