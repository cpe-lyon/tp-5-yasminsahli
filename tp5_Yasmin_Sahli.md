Yasmin SAHLI 
3ICS 
# TP 5 Systèmes de fichiers, partitions et disques

## **Exercice 1 : Disques et partitions**

1. *Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement
alloués ; puis démarrez la VM*

Clique droit sur la machine, settings, add, hard disk, 5 Go, 

2. *Vérifiez que ce nouveau disque dur est bien détecté par le système*

On fait ```lsblk ``` et on voit en bas de la liste notre disque de 5 Go.

3. *Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83),
et une seconde partition de 3 Go en NTFS (n°7)*

On fait ``` sudo fdisk -l``` pour avoir des informations sur les partitions déjà existente. 
Ensuite on fait ``` sudo fdisk /dev/sdb``` pour pour sélectionner le disque que l'on veut partitionner.
On est désormais dans fdisk, on fait : 

n pour nouvelle partition 
p pour choisir le type primaire 
1 pour le numéro de la partition 
1er secteur valeur par defaut en faisant entree (on peut pas commencer avant 2048 car avant cela il y a la table de partition) ( table de partition limité à 1 Mo)
last secteur : +2G

on obtient : 

``` Created a new partition 1 of type 'Linux' and of size 2 GiB ```

Pour la deuxième partition on fait  :

n pour nouvelle partition 
p pour choisir le type primaire 
2 pour le numéro de la partition 
1er secteur valeur par defaut en faisant entree
2eme secteur valeur par defaut en faisant entree, qui prend les 3Go restant sans qu'on ai besoin de le préciser 

on obtient : 

``` Created a new partition 1 of type 'Linux' and of size 3 GiB ```

J'ai pas mis le bon type pour la deuxième partition donc on fait :  
t 
2 pour le num
7 pour le numéro de NTFS 

on fait p pour vérifier et on a bien la deuxième partition en HPFS/NTFS/exFAT

on fait w pour enregistrer

4. *A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers.
A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)*

On sait que ma partition 1 est en Linux donc elle doit se formater avec ext4, on fait : 
``` 
sudo mkfs.ext4 /dev/sdb1
```

Pour la deuxième partition est en NTFS donc elle doit se formater avec ext2, on fait 

```
sudo mkfs.ntfs /dev/sdb2
```


5. *Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?*

la commande ``` df -T ``` nous donnes le type de système de fichiers. Or on a pas encore monté (on a donc pas encore relié le système de fichier de la partition avec celui de la machine), donc le système de fichier du deuxième disque ne peut pas être affiché.

6. *Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la
machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des
UUID en raison de l’impossibilité d’effectuer des copier-coller)*

(ne pas mettre le UUID car trop long mais mettre le nom de partition) 

on fait : 
``` 
sudo nano /etc/fstab
```
et quand on est dans le fichier on le ajoute les deux lignes suivantes : 

```
/dev/sdb1   /data   ext4    defaults    0   0
/dev/sdb2   /win    ntfs    defaults    0   0
```

7. *Utilisez la commande mount puis redémarrez votre VM pour valider la configuration*

``` 
sudo mount -a

reboot
```


9. *Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer
les Additions invité de VirtualBox).*

## **Exercice 2 : Partitionnement LVM**

1. *On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de
fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes
du fichier /etc/fstab*

Pour démonter un système de fichiers monté, on utilise la commande sudo umount nom_systeme. Dans notre cas, on démonte les systèmes de fichiers /data et /win. Puis on supprime les lignes correspondantes du fichier /etc/fstab pour éviter que les systèmes de fichiers soient automatiquement remontés.

2. *Supprimez les deux partitions du disque, et créez une partition unique de type LVM*

Pour supprimer une partition, il faut utiliser la commande sudo fdisk chemin_partition. Dans notre cas, on tape la commande sudo fdisk /dev/sdb. Puis on tape d pour delete la partition que l'on veut supprimer. 

3. *A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en
utilisant la commande pvdisplay.*

Pour formater le disque physique et l'intégrer au système de gestion on fait : 
```
pvcreate /dev/sdb1
```
Pour vérifier si cela a fonctionné on fait : 
```
pvdisplay
```

4. *A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que
le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.*

On fait : 
```
vgcreate vg00 /dev/sdb1
```
(vg00 est le nom du groupe, il est nécessaire)
Pour vérifier si cela a fonctionné on fait : 
```
vgdisplay
```

5. *Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.*

6. *Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans
l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data*

7. *Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez
la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.*


8. *Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de
volumes*


9. *Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas
oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.*

