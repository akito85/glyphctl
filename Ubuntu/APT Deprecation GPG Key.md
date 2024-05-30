Import Public Keys
```
wget -qO - https://keys.anydesk.com/repos/DEB-GPG-KEY | sudo gpg --dearmour -o /usr/share/keyrings/microsoft.gpg
```
List Existing Keys
```
sudo apt-key list
```

Export Keys
```
sudo apt-key export BE1229CF | sudo gpg --dearmour -o /usr/share/keyrings/microsoft.gpg
```
**Note:** The `BE1229CF` value comes from the last 8 characters of the `pub` code.

Update Key
```
deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/edge/ stable main
```

Update APT
```
sudo apt update
```

Remove the Key
```
sudo apt-key del BE1229CF
```

------
Another Method
```
sudo cp trusted.gpg trusted.gpg.d
```

Refresh Key Method
```
gpg --refresh-keys
```

Automated Method
```
sudo apt-key list 2>&1 | grep -E '(trusted.gpg.d)' -A 3 | grep -v '^\-\-' | grep -v '^pub ' | sed 's@.*/trusted.gpg.d/\(.*\)@\1@g' | awk 'NR%2{printf "%s ",$0;next;}1' | awk '{print "sudo apt-key export "$10$11" | sudo gpg --dearmour -o /usr/share/keyrings/"$1}' | xargs -I{} eval("{}")
```

```
`sudo apt-key list 2>&1 | grep -E '\/(trusted.gpg.d)' -A 3 | grep -v '^\-\-' | grep -v '^pub ' | /bin/sed 's@.*/trusted.gpg.d/\(.*\)@\1@g' | /bin/awk 'NR%2{printf "%s ",$0;next;}1' | /bin/awk '{print "sudo apt-key export "$10$11" | sudo gpg --dearmour -o /usr/share/keyrings/"$1}' | xargs -I'{}' bash -c "eval '{}'"`
```

```
`sudo apt-key list | sudo awk -v n=4 'n==3{k=$(NF-1)$NF;cmd="apt-key export "k"|gpg --dearmour -o "d;print cmd;system(cmd)}/^\/.*\/trusted\.gpg\.d\//{d=$1;n=0}{n++}'`
```