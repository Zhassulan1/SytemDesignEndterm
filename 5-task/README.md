# 5-Task. e-Voting
Assumed that most of the population has Digital signature (ЭЦП) keys to sign their vote.

## Authentication

1.  Biometric authentication: requesting egov to authenticate users. EGov may not be able to handle that
    1. Scale egov
    2. Ask user to authenticate beforehand using egov, and later use Digital signature
2. Digital signature
    
    Probably does not require much computational resources
    

### Non-Functional requirements

1. 100M DAU (12 hour day) = 2300 RPS on average. Peak = 20000 RPS
2. Security as a priority
3. Anonymity
4. Votes impossible to change

### Functional Requirements

1. Voting
2. Geo sharding, dividing votes based on the home town of voter
    
    (might be useful for countries that have system like USA)
    
3. Anonymity

### CAP theorem

Consistency has more priority than availability. Because we have to make sure that every voter’s voice is saved. By prioritizing availability, we may introduce problems with synchronization.

### Trade offs

1. Performance and verifying, by introducing mechanisms to make sure that message/token is received, we add potential latency
2. Security, dependence and performance, by making our app more secure by digital signatures, I added dependence on eGov like service, to verify, authenticate user, also this verification may reduce performance in case of biometry.
3. Anonymity and reality. I could add ring signature technology to hide voter completely. But I couldn’t understand how it works. In implementation, this technology would be even more difficult.

### Anonymity

If there is a demand for top-tier anonymity, we probably should use ring signature (very difficult to even understand). I have spent too much time trying to understand it. Failed!

Implementation would hide voter among thousands of voters

https://en.wikipedia.org/wiki/Ring_signature

https://eprint.iacr.org/2025/002.pdf

But instead of focusing on blockchain, I decided just to not store information that would connect user to vote:

### Brief overview:

1. User authenticates
2. Server sends one time token and city where request should be sent. 
3. User verifies that token was received
4. If user doesn’t verify just won’t be on db
5. After user verifies, system writes token to separate voting db as active, and to main db changes user status as “received token”.
6. User makes selection, sends vote.
7. API Gateway receives, re-routes to voting system load balancer
8. Load balancer send to required city’s load balancer
9. Load balancer sends to voting system.
10. After vote is inserted to db, token is deleted

Database Choice: postgres because transactionality is required

All interactions happen in encrypted channel (HTTPS)

There is also counting system, that counts insertion numbers for each candidate, after voting day

![](e-voting.svg)