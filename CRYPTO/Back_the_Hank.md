# BackTheHank

The Kindasus Bank prioritizes the security of its users by implementing two separate databases to store partial user information.
However, the bank has introduced two distinct login methods, which is kinda sus.
Embark on a journey through the bank's website!

http://chall.ctf.k1nd4sus.it:31337

## Writeup

The website's homepage displays the top clients of Kindasus Bank. Upon closer inspection of the Network tab, one can identify the endpoint:

chall.ctf.k1nd4sus.it:31337/api/top-clients/MS01

with the infamous MS01. By base64-decoding "MS01" one can see that it becomes "1-5", by modifying that input one can dump the whole database. A limit of 50 had been set in order to limit big requests, therefore going by 1-50, 51-100 and so on would have been a way to proceed, but "0-0-0" (best locket's password ever, isn't it?) and similar formatted strings, encoded in base64, were already providing access to the entire dump (1992 users).

Once the partial user information would have been recovered, one should have utilized the "Forgot Account Data?" option on the login page. This option would respond with "If the provided information is correct, recovery instructions will be sent." unless the correct information would have been indeed provided. By following this process, the flag could be obtained as a response.

At this juncture, there were two possible paths to pursue: password cracking or RSA module factorization. Fortunately, the vulnerability at hand was a fundamental one: two RSA modules shared a common factor, and for one of these modules, the username was also included in the partial information that had been previously recovered. By executing a script based on the greatest common divisor (GCD) across all the RSA modules, one could have identified both pieces of information necessary to conquer the challenge and acquire:

KSUS{H4NK_B4CK3D_U_RSA_CLU3v3R}