# Datacenter Spine-Leaf NetDevOps

Projet de fin d'etudes BUT 3 Reseaux & Telecommunications : simulation et automatisation d'une architecture datacenter Spine-Leaf avec Containerlab, FRRouting et Ansible.

## Architecture

- 2 spines sous FRRouting, AS 65000
- 3 leafs en eBGP underlay, AS 65011 a 65013
- EVPN-VXLAN avec les VNIs L2 10010 (production), 10020 (developpement) et 10030 (AAA)
- Un firewall Linux entre les segments, avec politique nftables restrictive
- Deploiement de la configuration routeur et des segments avec Ansible

Le plan d'adressage detaille est defini dans `ansible/host_vars/` et la topologie dans `topology.clab.yml`.

## Prerequis

- Docker Engine Linux accessible depuis le terminal utilise pour Containerlab et Ansible
- Containerlab 0.82 ou version compatible
- Ansible avec le plugin de connexion Docker

Utiliser le meme contexte Docker pour Containerlab et Ansible. Si Containerlab est lance avec `sudo`, conserver le meme contexte pour Ansible.

## Deploiement

Depuis la racine du depot :

```sh
containerlab deploy -t topology.clab.yml
ANSIBLE_CONFIG=ansible/ansible.cfg ansible-playbook -i ansible/inventory.yml ansible/deploy.yml
```

Le playbook verifie la presence des interfaces Containerlab avant de modifier les routeurs. Si cette verification echoue, corriger d'abord le deploiement et l'acces de Containerlab aux namespaces reseau.

## Verification

```sh
containerlab inspect -t topology.clab.yml
docker exec clab-but3-datacenter-spine1 vtysh -c 'show bgp summary'
docker exec clab-but3-datacenter-leaf1 vtysh -c 'show bgp l2vpn evpn summary'
docker exec clab-but3-datacenter-leaf1 vtysh -c 'show evpn vni'
```

La configuration actuelle fournit une segmentation VLAN/VXLAN et le routage inter-segments via le firewall. Elle ne configure pas encore de VRF/L3VNI, et le conteneur AAA doit encore recevoir FreeRADIUS.

## Fichiers generes et secrets

Le repertoire `clab-but3-datacenter/` est genere par Containerlab et ignore par Git. Les cles privees, certificats et fichiers locaux de credentials sont egalement exclus. Ne jamais ajouter de secret au depot.