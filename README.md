# vagrant-k3s - UBUNTU
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
[![Vagrant](https://img.shields.io/badge/Vagrant-1868F2?style=for-the-badge&logo=Vagrant&logoColor=white)](none)
[![VirtualBox](https://img.shields.io/badge/VirtualBox-21416b?style=for-the-badge&logo=VirtualBox&logoColor=white)](https://www.google.fr)
![Shell Script](https://img.shields.io/badge/Shell_Script-121011?style=for-the-badge&logo=gnu-bash&logoColor=white)
![image](https://github.com/user-attachments/assets/93049f18-16ff-46a9-a4ef-029a020cc2c5)

## Overview

Project status: OK

Ce projet vise l'installation d'un cluster complet k3s avec un nombre de worker à definir. (Penser a configurer le Vagrantfile en conséquence).
Tout est automatisé, le master utilise un Taint pour n'avoir de Pods que sur le/les workers.

## Prérequis :
+ Virtualbox
+ Vagrant

---

## Utilisation
```ruby
vagrant up
```

## Ajout workers

Changer NodeCount dans le Vagrantfile
```ruby
Vagrant.configure(2) do |config|

  # Change to add more workers
  NodeCount = 1
  
```
