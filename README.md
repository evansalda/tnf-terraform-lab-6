# CREATION DU SERVEUR WEB

## Déclaration des datasources

Plutôt que de directement renseigner l’ID du VPC et des sous-réseaux utilisés par vos ressources, vous allez utiliser des datasources pour les récupérer à partir de leur nom.

Dans le répertoire nuumfactory-labs/main-lab, crééz un fichier nommé **datasources.tf** et déclarez-y les 3 datasources suivantes :

- Récupération des données du VPC dont le tag **Name** a pour valeur "nuumfactory-vpc"
- Récupération des données du sous-réseau dont le tag **Name** a pour valeur "nuumfactory-public-subnet-1"
- Récupération des données du sous-réseau dont le tag **Name** a pour valeur "nuumfactory-public-subnet-2"

Une fois vos datasources déclarées, utilisez la première pour renseigner l’ID du VPC de vos security groups.

Supprimez la variable **vpc** qui n'est plus utile.

## Déclaration de la VM EC2

En vous appuyant sur la [documentation officielle de terraform](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance), déclarez votre serveur web dans un fichier nommé ec2.tf avec les caractéristiques suivantes :

| Caractéristique                                | Valeur                                     |
|------------------------------------------------|--------------------------------------------|
| ID de l’ami                                    | ami-074e262099d145e90                      |
| type d’instance                                | La valeur de la variable ec2_type          |
| sous-réseau                                    | L’ID du sous-réseau public de votre choix* |
| Security group                                 | L’ID de votre security group ec2           |
| Association d’une adresse IP publique          | Oui                                        |
| Nom de la clé utilisée pour l’authentification | "nuumfactory-ec2-key-pair"                 |

*Utilisez la datasource adéquat pour renseigner cette valeur.

Enfin, ajoutez le tag **Name** en lui attribuant la valeur nuumfactory-\<env\>-ec2-\<digit\> (utilisez la variable locale adéquate).

## Déclaration des user datas

Sur AWS, les [**user data**](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) sont une liste d’instructions exécutées au lancement d’une VM EC2. Ces instructions peuvent être des commandes shell ou des directives cloud-init et sont listées dans un texte brut ou dans un fichier.

Les user data étant exécutés avec l’utilisateur root, il n’est pas nécessaire d’y utiliser la commande sudo.

Dans terraform, les user data sont renseignés au travers du paramètre **user_data** de la ressource **aws_instance** :

**Texte brut**

```
resource "aws_instance" "serveur_web" {
 .
 .
 .
  user_data = <<EOF
#!/bin/bash
echo "mes superbes user data !"
EOF
}
```

Notez que le contenu des user data n’est pas indenté, il se situe syntaxiquement au même niveau que la déclaration du block **resource{}**.

**Fichier**

```
resource "aws_instance" "serveur_web" {
  .
  .
  .
  user_data = file("user_data.sh")
}
```

Avec le fichier user_data.sh présent dans le répertoire où le code terraform se situe.

La VM EC2 que vous avez précédemment déclarée fera office de serveur web dans votre infrastructure. Vous utiliserez les userdata pour y installer et configurer la stack apache-php-mariadb dessus (en remplaçant **serveur_web** par le nom renseigné dans votre code) :

```
resource "aws_instance" "serveur_web" {
  .
  .
  .
  user_data = <<EOF
#!/bin/bash
sudo dnf update -y
sudo dnf install -y httpd php php-mysqli mariadb105
sudo systemctl start httpd
sudo systemctl enable httpd
usermod -a -G apache ec2-user
chown -R ec2-user:apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;
EOF
}
```

## Création du serveur web

Une fois votre serveur web déclaré, exécutez la commande **terraform fmt** puis la commande **terraform plan -var-file="developpement.tfvars"** : Corrigez les éventuelles erreurs obtenues et réexécutez la commande jusqu’à ne plus en obtenir.

Observez le plan d’exécution et s’il correspond à ce que vous souhaitez réaliser, exécutez la commande **terraform apply -auto-approve -var-file="developpement.tfvars"**.

## Test de connexion SSH au serveur web

Créez le fichier ec2-private-key.pem dans votre répertoire de travail et collez-y le contenu suivant :

```
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQB91BGbUmRV+bkF5Gnu1j0zSbNuLy/s/2JsVKhz0OIRY7mT7ESk
j1yxb/32g0SKXgrcY4IIYlOjx9GMd8iuaFShh6Nbv8GSgHm3z8AwLkHgY6+WXati
ckRBnEDKDbQ8F8K3xRBzJeM0s7hGkM3j4wQiLS5RBDL4nJeMdXekkn+HWKkjHEQ4
a6X27wSDklRONzFxyGSWLZ3cm9NrUNTRE9P2DoGMRC/Z+NloS4DRmJQvJSykS4pp
+YO2rz0dT03kHrk40jkkM83rHCn9ELb86nRWHUsCk9PtC8RetTQbt4/xN5CB+OgM
MKg+jxPZupv73QWRpXx2pCjeIsS47yWPI9u1AgMBAAECggEAS5a8RPyH/gYYmmuP
H8Vf2pGp0sVSGyOIMt/gmkKfrCamczB6RAlDe+x1OkO9RwobqC23DeZTrI37WlET
I4LVZHwhLJrTZHj9peiN4ePH+06nSsNWk7tlOazuVvNIlNkJRnCB40qdZSmZx/px
VTcpYoaVzmGhZSxc9ioTB7BiICHRJpISuovtYhyHic/jtf0QvnOXhDV2FcPLeB8G
86jWB+KiTrlIQS5T/gPtFc/lGzL5jsix9WfDU1C/uW4RntXaszr1RyEnZKAjjryh
j8ucuZizH2MBMyHADYBIym1p/PIPckNp/Vfl9bmlZLDGDoo0rCJ+K/Hi7kBFzgub
UPii+QKBgQC5rDRTBb9O9UVpsSZFS/Hyp/MH1G+SDOAhuVTZ/OXwlydhbPsm1BZq
xqQFIhGV038BdFHb3QHNrPhBv1HHmLDPGgIInRx154vQJgN34i57HqkxW7O+F8Jg
GW+sfy0c4zROgOf63GkuHdDivdrzikBA/SshHqzeZn+BphRkjLsl5wKBgQCtfQyi
IzgdvvluQJBOYdaBIRFQlejlJzRU9ViX243q9fm/RDVq6ETLA/ijnLdJnB0NzE4A
E8cHfkAjFS16xDi6UhFGgVpuq3WPT2kBsV/iCL5xsafPQUxwq0d91xmn521V7OqC
NDpDHrQoaA4KlSp16V2qikgBkwtJiPSFSZkGAwKBgFMQRgxKSu7A7X++H7fqpOAA
4MnE8PDuz6pmph4rdJbwmE6OmcEiKrE0Epa1Sha0GmKFLkXlFnR0CFApjiV0Gs1b
/kLqPpxErRi+mNieGFs+OUT6mGvXZz7kwj/yWTVOM81W//ELgAaAkj2N4BEJ7Xrl
h9D2TzHjuvE+YmslRmhLAoGBAKk38/6iQ7Yf9MOpjhgmLkg9rNnhnw0FNHI57XQR
31dzHWuGaGQishcjhH5x+gV+lIhE40AICnYwmvadTYMVqg9yxQ70VPTloQFr/4x7
Kn8a8EeNdZUeqCStrEn+aTPw9CB/ui3OK5YUeL2A4VFJNeVU/tu9jYabmsLbJ0Zr
BytpAoGAD1MJ7uA++/8PpX3pNpLjROLXbiu2b9u0KEkut5QHLUNJ3gcMHI/rmlSn
rEyVSBT36+7oQ3i5jsObhMikMOvSbzYkNH6jNcyRHjDDaFQ6VpxLyVYwQas90v13
1S62VWRyK1J9qg3m7Y2x8mHEdaonWnwVoVkrPqIY9Uxp74hbhT8=
-----END RSA PRIVATE KEY-----
```

Exécutez ensuite la commande suivante en remplaçant **adresse-ip-publique** par l’adresse IP publique de votre serveur web, consultable sur la console AWS :

```
ssh -i "ec2-private-key.pem" ec2-user@adresse-ip-publique
```

Répondez **yes** à la question **Are you sure you want to continue connecting (yes/no/[fingerprint])?** : Vous êtes maintenant connecté à votre serveur web.

Tapez la commande **exit** pour vous déconnecter de votre serveur.