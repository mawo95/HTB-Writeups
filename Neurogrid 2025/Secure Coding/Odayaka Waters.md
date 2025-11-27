**Idea**
Patch the insecure param validation which allows to claim admin role

**Code**
```$user = User::create([
            'name'     => $_REQUEST['name'],
            'email'    => $_REQUEST['email'],
            'password' => Hash::make($_REQUEST['password']),
            'role'     => $_REQUEST['role'] ?? 'user',
        ]);
```

Replace it with this secure version:
```
$user = User::create([
            'name'     => $_REQUEST['name'],
            'email'    => $_REQUEST['email'],
            'password' => Hash::make($_REQUEST['password']),
            'role'     => 'user',
        ]);
        ```
