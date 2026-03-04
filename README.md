# proxmox_ansible_sdn_vm_maker

Ansible'i playbook Proxmoxi virtuaalmasinate kloonimiseks õpilasrühmale CSV-failist.
Iga õpilane saab oma virtuaalmasina, SDN-võrguliidesega ja `PVEAdmin` õigused (PVEAdmin muuta olenevalt keskonnast).

**Originaalne bashi skripti autor:** Valdo Nõlvak  
**Ansible ümberkirjutuse autor:** joosep Alasoo IT-23


## Mida see teeb

Iga CSV-faili rea kohta playbook:

1. **Kloonib** malli VM-i (`qm clone`) ja nimetab selle `<lab>-Server-<group>-<EesnimiPerenimi>`
2. **Seab võrgu** liidese järgmisele SDN-sillale (`vnet20`, `vnet21`, …) `virtio` draiveri ja tulemüüriga
3. **Annab** õpilasele `PVEAdmin` õigused tema VM-il läbi `pveum aclmod`

---

## Nõuded

| Nõue | Märkused |
|---|---|
| Ansible ≥ 2.10 | Juhtmasinal |
| `community.general` kollektsioon | Paigalda `requirements.yml` kaudu |
| SSH-ligipääs Proxmoxi hostile `root`-ina | `qm` ja `pveum` vajavad root-õigusi |
| Proxmox VE koos seadistatud SDN-iga | SDN tsoonid/vnetid peavad enne käivitamist olemas olema |

Paigalda vajalik kollektsioon:

```bash
ansible-galaxy collection install -r requirements.yml
```

---

## Projekti struktuur

```
proxmox-vm-deploy/
├── clone_vms.yml               # Peamine playbook
├── requirements.yml            # Ansible Galaxy sõltuvused
├── students.csv.example        # Näidis-CSV (kopeeri → students.csv)
├── inventory/
│   └── hosts.yml               # Proxmoxi hosti ühenduse andmed
├── group_vars/
│   └── proxmox.yml             # Kõik muudetavad muutujad
└── roles/
    └── proxmox_clone/
        ├── defaults/main.yml   # Rolli vaikimisi muutujad
        ├── tasks/main.yml      # Kloonimine / võrk / ACL ülesanded
        └── meta/main.yml       # Rolli metaandmed
```

---

## Kiire alustamine

### 1. Klooni repo

```bash
git clone https://github.com/<sinu-org>/proxmox-vm-deploy.git
cd proxmox-vm-deploy
```

### 2. Seadista Proxmoxi host

Muuda `inventory/hosts.yml`:

```yaml
pve1:
  ansible_host: 192.168.1.10   # sinu Proxmoxi IP
  ansible_user: root
```

### 3. Häälesta muutujad

Muuda `group_vars/proxmox.yml` vastavalt oma keskkonnale:

| Muutuja | Vaikeväärtus | Kirjeldus |
|---|---|---|
| `template_vm_id` | `121` | Kloonitava malli VM ID |
| `base_vm_id` | `100` | Kloonitud VM-ide algne ID |
| `base_vlan` | `20` | Algne SDN vneti number |
| `lab_label` | `Win` | Sildieesliide VM nimes |
| `group_label` | `IT23` | Õpilaste rühm VM nimes |
| `user_domain` | `hkhk.edu.ee` | Proxmoxi autentimisrealm |
| `proxmox_role` | `PVEAdmin` | Õpilasele antav roll |

### 4. Valmista CSV ette

Kopeeri näidis ja täida õpilaste andmetega (ilma päisereata):

```bash
cp students.csv.example students.csv
```

Vorming: `eesnimi,perenimi,kasutajanimi`

```
Jaan,Tamm,jtamm
Mari,Kask,mkask
```

### 5. Käivita playbook

```bash
ansible-playbook clone_vms.yml -i inventory/hosts.yml \
  --extra-vars "csv_file=students.csv"
```

---

## Proovikäivitus (kontrollrežiim)

```bash
ansible-playbook clone_vms.yml -i inventory/hosts.yml \
  --extra-vars "csv_file=students.csv" --check
```

> Märkus: `qm` ja `pveum` käsud jäetakse kontrollrežiimis vahele, kuna need käivitatakse kaughostil.

---

## VM-i nimetamise konventsioon

```
<lab_label>-Server-<group_label>-<Eesnimi><Perenimi>
```

Näide vaikeväärtustega ja õpilase `Jaan Tamm` puhul:

```
Win-Server-IT23-JaanTamm
```

---

## Turvasoovitused

- **Ära kunagi lisa päris `students.csv`** giti — see on lisatud `.gitignore`-i
- Kasuta **Ansible Vault**-i, kui pead salasõnu salvestama:  
  `ansible-vault encrypt group_vars/vault.yml`
- Proxmoxi root SSH-võti ei tohiks repositooriumis olla

---

## Litsents

MIT
