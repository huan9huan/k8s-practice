## kubectl apply

## test to send some words
```
curl -XPOST 'http://127.0.0.1:31244' -H 'Content-Type: application/json' -d'
{
    "Say" : "Hello world!"
}
'
```