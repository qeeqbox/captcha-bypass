<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/captcha-bypass/main/captcha-bypass.png"></p>

A threat actor may bypass the Completely Automated Public Turing test to tell Computers and Humans Apart (captcha) by breaking the solving logic, human-assisted solving services, or utilizing automated technology.

## Example #1
1. Threat actor fills up a feedback form with the wrong captcha
2. Server sends a request to answer the captcha correctly
3. Threat actor fills up a feedback form with null 
4. Sever does not handle null properly and continues to process the request

## Impact
Vary

## Risk
- Perform unauthorized action

## Redemption
- Different captcha
- Device fingerprinting

## ID
d9d7a4e5-dfa6-4d7a-a5c2-65799113437d

## References
- [perimeterx](https://www.perimeterx.com/resources/blog/2020/captchas-hard-for-humans-easy-for-bots/")
