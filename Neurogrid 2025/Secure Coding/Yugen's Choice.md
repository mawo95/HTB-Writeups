**Idea**
Stop command injecting 

**Code**
The insecure usage of pickle alles code execution
```
def _unpickle(b64_data: str) -> Any:
    decoded = base64.b64decode(b64_data)
    return pickle.loads(decoded)
```
```
pickle.loads
```
executes the command in shell!
**Solution**
Full replacement of pickle via normal base64  decoding
