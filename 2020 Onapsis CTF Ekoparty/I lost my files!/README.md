# I lost my files!
```
I lost three of my files!!! I only remember the next few things:

    One of the files content, is the word onapsis repeated. 'onapsisonapsisonapsisonapsis'. But i cant remember the length of the content.
    One of the files content, is a repeated character. Example 'oooooooooooooooooooooooooooooooo'
    One of the files content, is a representation of a number. Example '123912983791273918273'

Oh, i almost forget it!!! All the files were ciphered using a stream cipher with the same keystream.

Can you help me with that???
```

The challenge mentions some files, and we are given a folder called *files*, which contains around 1000 files, of 31Kb each. It also mentions that the files were ciphered with a stream cipher, and that there are three files whose content format is kind of known. And the most important part, the keystream used along all the files was the same.

A stream cipher is just a way of encrypting data, that consists of one simple step. We have the plain text that we want to encrypt, and a key stream that is basically a string that has the same length of the plain text. And the algorithm itself consists of doing an *XOR*, between each one of the plain text characters and the corresponding one in the keystream, and this is the encrypted string.

The important thing to notice here is that *XOR* operation can be reversed. In our particular case, we have files that were encrypted with the same key, so for two different files we have the following:
```
file1 XOR keystream = file1_enc
file2 XOR keystream = file2_enc
```
So, if we were to XOR these two results we would get:
```
file1_enc XOR file2_enc 
= (file1 XOR keystream) XOR (file2 XOR keystream)
= (file1 XOR file2) XOR (keystream XOR keystream) <-- XOR is commutative
= file1 XOR file2
```
The result is basically an *XOR* of the initial content of both files. The only thing that we are missing here is if there is a way to **guess** somehow the content of some file, or to bruteforce it if possible. For this, we can go back to the initial information about this challenge, and notice that it says:
>One of the files content, is the word onapsis repeated. 'onapsisonapsisonapsisonapsis'. But i cant remember the length of the content.

This is perfect, because as the files have 31Kb, we can guess how many times the *onapsis* word will be repeated in one of the files, and that is: 
> onapsisonapsisonapsisonapsisona

After realizing this, the challenge is basically solved. We just need to make a script that tries this process we described before, take two files, *XOR* their content, and then *XOR* it with this last string. If one of both files that we selected is the one that has the onapsis word, we will manage to decrypt the other one. The following code does this process:

```python

from os import listdir
from os.path import isfile, join
import base64

def get_bytes_from_file(filename):  
    return bytearray(open(filename, "rb").read())

def xor(a,b):
    return bytearray([_a^_b for _a,_b in zip(a,b)])

#read all file names from the files folder
onlyfiles = [join("files", f) for f in listdir("files") if isfile(join("files", f))]

def decrypt_files(onlyfiles,bc):
	for i in range(0,len(onlyfiles)):
	    b1 = get_bytes_from_file(onlyfiles[i])[:31] #get encrypted data from file 1
	    for j in range(i+1,len(onlyfiles)):
		 b2 = get_bytes_from_file(onlyfiles[j])[:31] #get encrypted data from file 2
		 p1p2 = xor(b1,b2) # XOR encrypted data of file 1 with file 2
		 num = xor(p1p2,bc) # XOR this result with the string "onapsisonapsisonapsisonapsisona"
		 res = ""
		 try:
		   res = num.decode("utf-8")
		 except:
		   res = ""
		 if res != "": #if the decrypted file string is valid, it may be one of the files we are looking for
		   print(i,j,res)

onapsis_s = bytearray("onapsisonapsisonapsisonapsisona")
decrypt_files(onlyfiles,onapsis_s)

```

After running this code, we get the file content of the other two files:

![alt text][flag]

[flag]: flag.png

And the final flag consisted of combining these 3 strings:
> string 1 : "onapsisonapsisonapsisonapsisona"

> string 2 : "..............................."

> string 3 : "1761284612498125371253187651230"
