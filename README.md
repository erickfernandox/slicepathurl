[![made-with-Go](https://img.shields.io/badge/made%20with-Go-brightgreen.svg)](http://golang.org)
<h1 align="center">SlicePathURL</h1> <br>

<p align="center">
  <a href="#--usage--explanation">Usage</a> •
  <a href="#--installation--requirements">Installation</a> •
  <a href="#--why-use-slicepathurl">Why use SlicePathURL?</a> •
  <a href="#--how-does-slicepathurl-work">How does SlicePathURL work?</a>
</p>

<h3 align="center">SlicePathURL slices a URL into directory levels to complement tools like Nuclei in searching for vulnerabilities in directories beyond the root of the URL.</h3>


## - Installation & Requirements:

```bash
> go install github.com/erickfernandox/slicepathurl@latest
```
OR
```bash
> git clone https://github.com/erickfernandox/slicepathurl.git
> cd slicepathurl
> go build slicepathurl.go
> chmod +x slicepathurl
> ./slicepathurl -h
```
<br>

## - Why use SlicePathURL?

Examples:

Sometimes, Nuclei may fail to identify a vulnerability in the root domain, for example, in https://example.com/. However, it is possible that vulnerabilities may exist in paths beyond the root domain, such as in https://example.com/path_one/. 

Below is a real example that was found:

```bash
echo "https://subdomain.example.com/"|nuclei -tags rce

[INF] No results found. Better luck next time!
```

```bash
echo "https://subdomain.example.com/extranet/"|nuclei -tags rce

[2023-01-01 00:00:00] [CVE-2017-5638] [http] [critical] https://subdomain.example.com/extranet/
```

An RCE vulnerability, CVE-2017-5638, was discovered in Apache Struts in an application hosted at https://example.com/extranet/, but it was not found in the root directory of https://example.com/.

Below are additional examples where SlicePathURL was used to identify vulnerabilities that were not located in the root directory of the domain, but rather in a subdirectory:

```bash
[CVE-2019-6802] [http] [medium] https://example.com/path_level2/%0d%0aSet-Cookie:crlfinjection=1; -> CRLF Injection
[CVE-2018-11784] [http] [low] https://subdomain.example.com/path_level2///interact.sh/%2F -> Open Redirect
[elmah-log-file] [http] [medium] https://xxx.example.com.br/perdiminhasenha/elmah.axd?AspxAutoDetectCookieSupport=1 -> Debug Information Exposed
[git-exposed] [http] [medium] https://xxx.example.com.br/path_level2/.git/config -> Git Exposed
```


## - How does SlicePathURL work?


```bash

subfinder -d example.com | gauplus | slicepathurl -l 2 > urls_all_paths_level2.txt
cat urs_all_paths_level2.txt | nuclei -tags crlf,rce,redirect
```

Indetificando Git Exposed em 3 niveis:

O slicepathurl pega uma URL e divide em 3 niveis:
https://example.com/level2
https://example.com/level2/level3

```bash
cat urls_all_paths_level2.txt | slicepathurl -n 3 | httpx -path /.git/config -mr "refs/heads"
```
https://example.com/level2/.git/config
https://example.com/level2/level3/.git/config
