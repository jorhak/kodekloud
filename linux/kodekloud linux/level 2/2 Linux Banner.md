```
During the monthly compliance meeting, it was pointed out that several servers in the `Stratos DC` do not have a valid banner. The security team has provided serveral approved templates which should be applied to the servers to maintain compliance. These will be displayed to the user upon a successful login.  
  

  

Update the `message of the day` on all application servers for `Nautilus`. Make use of the approved template located at `/home/thor/nautilus_banner` on the jump host.
```
# Ver banner
```bash
cat nautilus_banner
```

```text
################################################################################################
  .__   __.      ___      __    __  .___________. __   __       __    __       _______.        # 
       |  \ |  |     /   \    |  |  |  | |           ||  | |  |     |  |  |  |     /       |   #
       |   \|  |    /  ^  \   |  |  |  | `---|  |----`|  | |  |     |  |  |  |    |   (----`   #
       |  . `  |   /  /_\  \  |  |  |  |     |  |     |  | |  |     |  |  |  |     \   \       #
       |  |\   |  /  _____  \ |  `--'  |     |  |     |  | |  `----.|  `--'  | .----)   |      #
       |__| \__| /__/     \__\ \______/      |__|     |__| |_______| \______/  |_______/       #
                                                                                               #
                                                                                               #
                                                                                               # 
                                                                                               #
                                 # #  ( )                                                      #
                                  ___#_#___|__                                                 #
                              _  |____________|  _                                             #
                       _=====| | |            | | |==== _                                      #
                 =====| |.---------------------------. | |====                                 #
   <--------------------'   .  .  .  .  .  .  .  .   '--------------/                          #
     \                                                             /                           #
      \_______________________________________________WWS_________/                            #
  wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww                        #
wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww                       # 
   wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww                         #
                                                                                               #
                                                                                               #
################################################################################################
Warning! All Nautilus systems are monitored and audited. Logoff immediately
```
# Ingresar a servidores
Abrimos otra terminal para estar mas comodos
```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```
# Crear banner
Agregamos lo que tenemos del primer comando que ejecutamos
```bash 
sudo vi /opt/banner_login
```
# Configurar ssh para que reconosca el banner
```bash
sudo vi /etc/ssh/sshd_config
```
#### Before
```text
#Banner none
```
#### After
```text
Banner /opt/banner_login
```
## Reiniciar servicio
```bash
sudo systemctl restart sshd
```
# Verificar
Nos salimos y volvemos a entrar al servidor.