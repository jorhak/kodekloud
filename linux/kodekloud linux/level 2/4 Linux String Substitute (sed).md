```
There is some data on `Nautilus App Server 1` in `Stratos DC`. Data needs to be altered in several of the files. On `Nautilus App Server 1`, alter the `/home/BSD.txt` file as per details given below:  
  

  

a. Delete all lines containing word `following` and save results in `/home/BSD_DELETE.txt` file. (Please be aware of case sensitivity)  
  
  
b. Replace all occurrence of word `the` to `is` and save results in `/home/BSD_REPLACE.txt` file.  
  
  
`Note:` Let's say you are asked to replace word `to` with `from`. In that case, make sure not to alter any words containing the string itself; for example up**to**, contribu**to**r etc.
```
# Ingresar al servidor
```bash
ssh tony@stapp01
```
# Eliminar todas las filas con la palabrea following
```bash
sudo touch /home/BSD_DELETE.txt
sudo chown tony:tony /home/BSD_DELETE.txt
sudo sed '/\bfollowing\b/d' /home/BSD.txt > /home/BSD_DELETE.txt
```
# Reemplazar the por is
```bash
sudo touch /home/BSD_REPLACE.txt
sudo chown tony:tony /home/BSD_REPLACE.txt
sudo sed 's/the/is/g' /home/BSD.txt > /home/BSD_REPLACE.txt
```