# Plus - misc
## Information about the challenge
- CTF Name : Hack.INI 2024
- CTF Organizers : Shellmates
- Category : misc / python exploitation
- Points : 492
- Solves : 8
- Onsite Players : 16
- Online Players : 94
- Flag Format : shellmates{flag}
- Date : 14/12/2023
- Duration : 36 Hours

## Description & Goal
We're given a plus (+) printer hosted at a server, and we're also given the source code of it:
```
ncat -v --ssl plus.hackini24.shellmates.club 443 
```


```python
#!/usr/bin/python3

secret="shellmates{F4ke_fL4g}"

class Plus:
    def print_plus(self):
        print("WELCOME to the most secure + printer")
        self.plus=input("give me how many + you want to be printed on the screen : ")
        text="printing {self.plus} +'s : \n{:+^"+self.plus+"}"
        print(text.format("",self=self))

a=Plus()
try:
    a.print_plus()
except:
    print('an error has occured ... \n')
```

Our goal is to find the vulnerability in the given python script and exploit it to find the flag.

## My stream of thought
### Understanding how the code works
First of all, we need to understand how this code works:

1. the flag is stored in the `secret` variable.
2. a `Plus` class is defined, containing the `print_plus` method.
3. in `print_plus`, the user is asked how many `+`'s they want to be printed out. this piece of information is stored in `self.plus`.
4. `self.plus` is concatenated into `text`, resulting in a string that can be formated by replacing `self.plus` with its value, and that's with the help of `format()` in the next line.
5. the formated string is printed out.
6. we call the method with `a.print_plus()` after constructing an object `a` with error exception.

After understanding how the code works, we can tell that the vulnerability must be in the `format` function, and that's after manipulating `text` with our input.

### Reading about the problem
After a bit of [reading online](https://www.geeksforgeeks.org/vulnerability-in-str-format-in-python/), we find that `format` is vulnerable in our context, and that's by letting the user to inject a custom formated string and allowing them to access sensitive data, and in our case, the flag.

### How is that?
In the example given in [GeeksForGeeks](https://www.geeksforgeeks.org/vulnerability-in-str-format-in-python/) above, they injected two object property names `people_obj.fname` and `people_obj.lname` through the input wrapped in curly braces
```python
"Avatar_{people_obj.fname}_{people_obj.lname}"
```
resulting in a string that can be formatted with the values of those two properties, allowing us to access sensitive information.
```python
"Avatar_GEEKS_FORGEEKS"
```

### Well, How can we access the flag?
The flag is stored in a global variable, which means we can access it through the `__globals__` [attribute]() of our function. in other words, if we call this in `print_plus()`'s body, we'll get the fake flag:

```python
>> print(self.print_plus.__globals__[secret])
"shellmates{F4ke_fL4g}"
```

### Let's try it on our source code
If we try to give `"self.print_plus.__globals__[secret]"` in the input directly, we get an error, why is that? well, after concatenation, `text` would contain this:
```python
"printing {self.plus} +'s : \n{:+^self.print_plus.__globals__[secret]}"
```
let's split it:

+ the first value will be replaced naturally
    ```python
    >>"{self.plus}".format("", self=self)
    "self.print_plus.__globals__[secret]}"
    ```
    which means the error occurs in the second half

+ ```python
    >>'{:+^self.print_plus.__globals__[secret]}'.format("", self=self)
    "ValueError: Invalid format specifier"
    ```

The problem occurs because `"{:+^" +self.plus+ "}"` expects `self.plus` to be an integer to generate that many `+`'s.

### How do we go around it?
We should find a way to isolate the injected string from `"{:+^}"`.

Simple. Instead of injecting `"self.print_plus.__globals__[secret]"`, we will inject `"}{self.print_plus.__globals__[secret]"` so we will end up with a `text` variable looking like this:
```python
"printing {self.plus} +'s : \n{:+^}{self.print_plus.__globals__[secret]}"
```

After executing the script we find the flag:
```python
"shellmates{F4ke_fL4g}"
```

### Let's apply this with the real printer
We just need to execute the command given in the terminal, it will allow us to send and receive data to and from the real printer.

```
ncat -v --ssl plus.hackini24.shellmates.club 443
```

if we give the injected string in the input, we will simply get the flag :
```
shellmates{AV01d_F0RM4T_$TrIng_MISTaKE$_1N_PYth0N!!!}
```


I hope you liked my write up :D
