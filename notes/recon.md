# Recon

\-- **Subdirs:**

```
ffuf -c -u http://IP/FUZZ -w ~/wordlists.txt -timeout 100 -t 100 -fs 0 -b 'cookies'
```

**-- Subdomains:**

```
ffuf -u http://domain.com/ -H "Host: FUZZ.domain.com" -w "$(wl)" -timeout 100 -t 1 -fs 0
```
