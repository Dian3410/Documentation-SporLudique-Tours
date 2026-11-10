# Administration SSH

- Configurer une ip basée dans le vlan management sur la deuxième carte réseau.

```sudo nano /etc/network/interfaces```

```
allow-hotplug ens19
iface ens19 inet static
        address 10.0.130.110
        netmask 255.255.255.0
```

- Mettre à jour la vm, puis installer SSH

```
sudo apt update
sudo apt install openssh-server
```

- Vérifier le status de ssh

```
sudo systemctl status ssh
(doit être enable et active running)
```

- Configurer une ip dans le même réseau que le vlan management, sur le poste d'administrateur puis connectez vous en SSH.

```ssh morgan@10.0.130.110```

- Administrer à distance avec SSH.