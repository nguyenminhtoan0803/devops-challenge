Provide your CLI command here:
grep 'TSLA' ./transaction-log.txt | awk '{print $1}' | xargs -I{} curl -s "https://example.com/api/{}" > ./output.txt