
# Bob's Revenge 
```
Bob got tired of always decoding messages sent to him, so he decided to write his own encoder. Unfortunately, Bob forgot to write a decoder and can't read his message. Can you help him out?
```

We are given a file which contains Bob's encoder and the encoded message that we have to decode somehow. As we open the file, we can observe the following code:
```python
from base64 import b64encode as b
from binascii import hexlify as h
from random import shuffle as u

message = b"e13ad530e70edf28e654ee1dd004b35eb97cc774955297509e2c967c9b4cab4e8050b7479055ef1ed061b4409b2ade2ae420d02bca22c520f910d101da68887c9d75a75ebe50a417c02efc42"

e = enumerate
r = bytearray

def encoder(f, s):
    x = b(f)
    a = []
    for i, j in e(x):
        k = [*[s]*(i + 1), *x[:i]]
        u(k)
        for n in k:
            j = j ^ n
        a += [j]
    return h(r(a))
```
We can divide it in two main things basically, the encoder itself that was used to encode the message, and the message after it got encoded.

The first idea that came up to me was to examine the encoder's code, and find a way to "undo" the encoding process somehow. So after looking at it, i found out a few things that it was doing:

1. It receives two parameters, the string *f* that has to be encoded, and a key *s* that is an integer in the range *0-255* used to encode the string. 
2. Encodes the string *f* in base64 and stores this result in x
3. Loops over a list of pairs, where each pair is (i,j) and *j* corresponds to the *i-th* character in x.
4. For each pair, it builds a list which has *i+1* times the key *s*, and then the first *i* characters of *x*. For example, if *i=3*, *x=abcd* and *s=23*, the list would like as follows: *[23,23,'a','b','c']*  
5. It then shuffles this last list and does an *XOR* with each element of it and the current *j* character.
6. Finally, it takes the result of doing all these *XORs* and stores it on a new list *a*
7. When the loop ends, it transforms the list *a* into a bytearray and then converts it into a *hex* string.

It's important to point out that the only difference between the current iteration and the previous one is that the list *k* used to *XOR* the current character has one more *s* at the beginning and one more character of *x* at the end. Another important thing to notice is that, despite having a shuffle of *k* and introducing some kind of randomness, the shuffle itself has no effect in the outcome. This is because each element is then *XORed* with the character *j*, and it really doesn't matter in which order they are *XORed*.

The second thing to point out is that the *XOR* operation can be reversed, so if we have *c = a XOR b*, and we want to obtain *a* back, we just do *c XOR b = a*. This is the key part for the solution, because it means that, if we actually know the list for an iteration, we can undo the process and get the *j* character back. As we saw before, we can build the list for the current iteration with the one of the previous iteration, so we just need to know how the initial list looks like (explained in a bit).

This is almost done, we know how to reverse the loop part, and the rest is just undoing simple operations, like decoding base64 and unhexing the hex string. The following code does the decoding process using all the things we just mentioned:
```python
from base64 import b64decode as bdec
from binascii import unhexlify as uh

e = enumerate
r = bytearray

def decoder(f, s):
    f = list(uh(f)) # inverse of h(r(a))
    a = []
    k = []
    for i, j in e(f):
        k = [s]+k # add to the left of the list the key s one time
        for n in k:
            j = j ^ n
        a += [j]
        k += [j] # add to the right the character j one time
    f = ''.join(chr(i) for i in a) #transform the list of numbers into a string, where each number is transformed into it's corresponding ASCII character
    return bdec(f).decode("utf-8") #decode back from base64 and decode the resulting string with utf-8 to see the correct characters
```

There is only one tiny detail missing, we actually do not know the key *s* that was used to encode the message. Luckily, the key *s* is in the range *0-255*, easily bruteforceable. We just have to do the following to find the current *s* and decode the message:
```python
#iterate over all possible values of the key s
for s in range(0,256):
    try:
        msg_decoded = decoder(message,s)
        print(msg_decoded)
    except:
       continue
```
After running this we get the decoded message:

![alt text][flag]

[flag]: flag.png

The flag is:

> my_N4m3_i5n't_4l1C3_bu7_1'lL_k3ep_l00k1nG_f0R_W0nD3rl4Nd