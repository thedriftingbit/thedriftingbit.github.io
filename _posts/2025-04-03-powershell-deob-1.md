---
layout: post
title: "PowerShell Deobfuscation #1"
author: <author_id> 
categories: write-up
tags: [cybersecurity,powershell,deobfuscation,malware,blog]
---

Malware authors will often attempt to hide the function of their code - one way that they do this is through obfuscation. These posts will be focused on a series of cybersecurity labs based on PowerShell script deobfuscation. The labs utilize Invoke-Obfuscation (you can read more about this [here](https://github.com/danielbohannon/Invoke-Obfuscation)) to alter the scripts, so these write-ups are not meant to be followed to a fault. Despite this, the concepts and techniques applied in solving these labs can be very helpful.

## Stage 1

This lab begins with the dropper.ps1 file. Opening it in the virtual machine's text editor, we can see that the obfuscated code is decimal-encoded.

![decimal encoded values in a powershell script](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Dropper1_Overview.jpg)

This snippet at the beginning of the code shows the usage of a dot sourcing operator - this is used to execute the proceeding script or command. But what script or command will it be executing?

![](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Dropper1_IEX.jpg)

Let's break this command down.

![](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Dropper1_IEX2.jpg)

The core of this command is the `$verbosepreference` predefined variable. The default value of this variable is `SilentlyContinue`. We can see that this variable is being cast to a string with the `[string]` command, and then two of that resulting string's characters are being printed - 1 and 3. Remember that string arrays are zero-indexed, meaning that the first character would be the 0th index.

For example, the 0th character of `SilentlyContinue` is the letter 'S.' That means that the first character is 'i,' and the third is 'e.'

Let's put all of that together:
`([string]$verbosepreference)[1,3]` returns
```
i
e
```
Then, we see that 'x' is being appended to that result, so the returned values of `([string]$verbosepreference)[1,3]+'x'` are:
```
i
e
x
```

Finally, these values are joined using the `-join''` operator, which removes the whitespace and concatenates the values into a single string - `iex`. This string is an alias for **Invoke-Expression** in PowerShell, which is a cmdlet that executes the proceeding string as a PowerShell command. In our case, that string will be whatever the obfuscated script decodes to.

![](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Dropper1_IEX2.jpg)

This is important to take note of because when you are analyzing malicious PowerShell code, Invoke-Expression is often used to "detonate" the script. By removing the dot operator and any tricky instances of Invoke-Expression, this detonation is prevented.

Let's take a look at the end of the obfuscated script:
![](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Dropper1_End2.jpg)

Here, we see the following command:
``` powershell
| ForEach-Object{([char][int]$_)}
```
The pipe ( | ) character indicates that all of the preceding obfuscated code will be used as the input for this command. Then, the **ForEach-Object** cmdlet performs a designated operation on each unique item in the collection - in our case, each decimal value. Within the brackets, we see `[char][int]`, which we can recognize from before as being ways of casting variables as different types, as well as `$_`, which is just shorthand for the current value being evaluated. All together, this tells us that each decimal value will be converted to an integer, then into its ASCII character representation. This is how the program decodes the next stage of the malware.

The virtual machine running the lab has a local version of CyberChef installed. This is a useful tool for decoding and encoding data, as well as performing various operations. The current version of CyberChef can be found [here](https://gchq.github.io/CyberChef/).

Since we have previously identified the method the PowerShell code uses to decode its dropper, we can place just the obfuscated portion of the script into CyberChef and select the 'From Decimal' recipe, making sure to select the comma delimiter. From there, CyberChef does the rest.

![cyber chef input and output](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Dropper1_CyberChef.jpg)

We can save this as stage2.ps1 and move on to analyze it further.

## Stage 2

The second stage of this obfuscated code looks somewhat like the first. Note the lack of any Invoke-Expression at the beginning!

![decimal encoded values in a powershell script](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage2_Overview.jpg)
There appear to be decimal values with ASCII characters interspersed. Let's check out the bottom of this code and see if there are any operations happening.

![](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage2-End.jpg)
We can see a string of commands separated by pipes, so we know that there are some operations being used as input and output here. Once again, we can break this command down into pieces to better understand it.

This is the first command:
```powershell
.split('WT:;CY}!Jd')
```
This command will split the preceding string - in our case, the obfuscated code - according to the characters contained in the quotes. This explains the random ASCII characters seen above. By executing this command, the obfuscated code will be split into decimal values similar to Stage 1.

This is the second command:
```powershell
| %{([char][int]$_)}
```
This pipes the output of the first command (remember, that will be the obfuscated code in decimal form) into another ForEach command. In this case, the % sign is used as a shortcut to indicate ForEach, similar to the dot operator for executing a command. We see `$_` again, which means that each decimal value will be cast first as an int, then as a character, exactly the same as in Stage 1.

The final command is:
```powershell
|.($shellid[1]+$shellid[13]+'x')
```
This should look familiar! Similar to Stage 1, this is a sneaky way of executing the preceding code using Invoke-Expression. However, instead of using the `$verbosepreference` variable, this time the code is using the `$shellid` built-in variable. This variable will always return `Microsoft.PowerShell`. As a result, the first and thirteenth index will be 'i' and 'e' respectively, meaning that this command will use a dot operator to call Invoke-Expression.

![](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage2_IEX2.jpg)

Now that we know how the obfuscated code is being decoded, we can return to CyberChef. First, we must split the obfuscated code according to the string we found. This can be done with the Find/Replace recipe set to Regular Expression mode. The string `[WT:;CY}!Jd]` will match each of the characters inside the brackets, and in my case I replaced them with a space.

![cyber chef input and output](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage2_CyberChef.jpg)

With that finished, we use the 'From Decimal' recipe as before on our new input to find the code for Stage 3:

![cyber chef input and output](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage2_to_Stage3.jpg)

## Stage 3

This code looks similar to the previous two stages. The beginning code will do the character conversion of the decimal values ahead of time.

![decimal encoded values in a powershell script](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage3_Overview.jpg)

The obfuscated code is still being decimal-encoded, but the PowerShell code at the end is a little different from what we have seen previously:

![](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage3_End.jpg)

Once again, we see the pipe operator and a couple commands. The first is: 
```powershell
| ForEach-Object{[char]($_ -bxor '0x09')}`
```
Remember that by the time this part of the script executes, these decimal values will have been converted to their character forms. Then, this command takes each ASCII value and uses the `-bxor` operator to transform that value further. The `-bxor` operator stands for Bitwise Exclusive OR, otherwise referred to as XOR. It is likely that the character converted script will be unreadable until it is XOR'd with this command.

The second comamnd is:
```powershell
| &((variable '*MDR*').name[3,11,2]-join'')
```
This command looks similar to some of the others we have seen - it is pulling out characters at specific indices from an array based off of a variable. Given that there are three characters being called, we can guess what this command might be doing. Once again, we see the `&` which is used to execute a command or function. The value inside of the parenthesis is admittedly a little confusing to me - it looks as though it is meant to say `Get-Variable '*MDR*'`. This command would search for all variables in the current PowerShell console that match the wildcarded string `'*MDR*'`. Executing this within the lab's virtual machine environment returns no results, but why?

We can investigate this further on our personal machine:

![](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage3_MDR.jpg)

The MaximumDriveCount variable refers to the number of drives that can be handled or defined in a given session. My understanding is that this variable would not be set in a lab's virtual machine environment, which is why it does not exist. However, we can see what the rest of the command would do if that variable did exist:

![](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage3_MDR2.jpg)

Again, we see the `iex` cmdlet being called in a sneaky way. In the case of the lab's virtual machine, however, this value would not execute because the MaximumDriveCount variable does not exist. Attempting to execute the code shown would throw a "Cannot index into a null array" error.

Now that we understand the operations being performed on the obfuscated code, we can return to CyberChef and use the 'From Decimal' and 'XOR' recipes to deobfuscate.

![cyber chef input and output with the lab's solution](https://raw.githubusercontent.com/thedriftingbit/thedriftingbit.github.io/refs/heads/main/assets/img/powershell-deob-1/Stage4_Solution.jpg)

This returns the plaintext code that contains the flag for this lab.