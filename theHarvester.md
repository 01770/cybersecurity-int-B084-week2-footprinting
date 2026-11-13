Footprinting with theHarvester 
task 1: theHarvester -d microsoft.com -l 1000 -baidu
Result: no emails, no hosts found 
Task 2: theHarvester -d microsoft .com -l 50 -b all
result: 44 hosts found (hudson Rock), 0 emails .souce that need API key  were skipped 
what I learned: public sources can reveal emails and subdomains without touching the target this tool is to see what an attacker could find 
