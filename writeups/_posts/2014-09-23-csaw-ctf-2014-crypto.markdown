---
layout: post
ctf: "CSAW CTF 2014"
title: "Crypto 300 (feal & cfbsum)"
---

feal
======

First, you had to recognize the algorithm used for the encryption. There are quite a few hints in the text and in the challenge name. The algorithm is in fact "Feal4". There is an other variant called "Feal6", but if you take a look at the encrypt method, you can see that there are only 4 rounds of encryption instead of 6.

What you had to also realize is that it's a slightly modified version of Feal4 that was used. The two differences are that the gBox has a rotation of 3 instead of 2 and that the fBox input is reversed (byte 0 is byte 3 and byte 1 is byte 2).

After that you have to apply an [already known cryptanalysis](http://www.theamazingking.com/crypto-feal.php), but with the slight modification of the algorithm. What this changes is that output differential is of 0x00000004 instead of 0x02000000.

It takes about one minute to two minutes to run the whole thing and it requires about 72 encrypted plaintexts. It allows you to recover the subkey and you can use this information to decrypt the message sent initially.

Flag : flag{FealMeBaby}

cfbsum
======

The first step to decrypt everything is to reverse the XOR part that includes the sum and previous value. To do this you have to do the following :

ciphers = [ ... array of all the value of the challenge ... ]

for i in range(c):
	new_cipher = []
	sum = 0
	prev = 0

	for j in ciphers[i]:
		new_cipher[i].append(j ^ sum ^ prev)

		prev = j
		sum  = (sum + j) % 256

	ciphers[i] = new_cipher

After this the challenge becomes the decryption a N-time pad. This is a well-known cryptography problem where all the plaintexts are XORed with the same pad/mask. To decrypt a N-time pad you first have to define a charset of acceptable character for the plaintext. In this case, it's english text so using "qwertyuiopasdfghjklzxcvbnm.0123456789 {}_,'&" is pretty much everything that we need for the charset. The other thing that we need to converge fast to the right solution is a list of acceptable word. Using any dictionary is fine, but make sure it doesn't contain too many small word that are barely never used. Also, as the decryption goes, you may realize that there are word missing such as "scallywag" and you can add those manually to your dictionary.


	# Start by building a reference array of word that are acceptable
	with open("dict.txt", "rb") as f:
		words = f.read().split("\n")
		
	english = {}
	englishf = {}

	for w in words:
		for i in range(len(w) + 1):
			english[w[:i]] = 1
		englishf[w] = 1

	charset = "qwertyuiopasdfghjklzxcvbnm.0123456789 {}_,'&"
	lc   = len(c)
		
	# Reverse of the XOR sum and XOR previous byte as explained before
	for i in range(lc):
		sum = 0
		nc = [c[i][0]]
		for a in range(1, len(c[i])):
			sum += c[i][a - 1]
			sum = sum % 256
			nc.append(c[i][a] ^ sum ^ c[i][a - 1])
			
		c[i] = nc


	# This loop is using r and poss as a stack instead of doing recursive call
	r    = [[0] for i in range(len(c))] # Current value
	poss = [list(range(256))]           # Possible character to test
	max = 0
		

	while (True):
		index = len(poss) - 1
		
		# When we have tried all the possible value and we couldn't
		# find anything we backtrack
		while len(poss[index]) == 0:
			poss.pop()
			for i in range(lc):
				r[i].pop()
			index -= 1
			
			if (index < 0):
				print("Not found !")
				exit()
		
		x = poss[index].pop()
		good = True
		
		# We check for all the encrypted plaintexts the result if the key 
		# at that index was "x"
		for i in range(lc):		
			b = c[i][index] ^ x
			prev = "".join(map(chr, r[i]))[1:]
			
			# We make sure result are only in the charset
			if not chr(b) in charset and not chr(b) == "I":
				good = False
				break
				
			# If we hit a separator, we check if the previous word is an 
			# english word
			if chr(b) in ". {},&":
				regr = re.findall(r"[\w']+", prev)
				if len(regr) == 0 or not regr[-1].lower() in englishf:
					good = False
					break
			# If we hit a letter, we check if there are possible word starting 
			# with the letters so far
			else:
				data = prev + chr(b)
				regr = re.findall(r"[\w']+", data)
				
				if not regr[-1].lower() in english:
					good = False
					break
		
		# If all the checks are good, we continue with the next index
		if good:
			for i in range(lc):
				r[i].append(c[i][index] ^ x)
			
			poss.append(list(range(256)))
		
		# For debug purposes, when we reach a point further, we print
		# the results so far.
		if index > max:
			max = index
			for i in range(lc):
				print("".join(map(chr, r[i]))[1:])
	

Flag : key{This is my pad there are many like it but this pad is my own}