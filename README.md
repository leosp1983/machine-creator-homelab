# machine-creator-homelab

Playbook Ansible (para rodar via AAP/Tower) que provisiona uma VM Linux completa no homelab: reserva IP no phpIPAM, cria a VM no vCenter, adiciona ao inventário dinâmico e configura o sistema (rede, RHSM, pacotes, timezone, ingresso no domínio AD).

## Pré-requisitos

- Ansible 2.10+
- Coleção `community.vmware`

## Variáveis obrigatórias (definir via env var ou credencial no AAP — nunca hardcode)

| Variável | Descrição |
|---|---|
| `PHPIPAM_TOKEN` | Token de API do phpIPAM |
| `VCENTER_USERNAME` / `VCENTER_PASSWORD` | Credenciais do vCenter |
| `VM_INITIAL_PASSWORD` | Senha inicial do `root` para conexão SSH na VM recém-criada |
| `RHSM_USERNAME` / `RHSM_PASSWORD` | Credenciais de registro no Red Hat Subscription Manager |
| `AD_JOIN_PASSWORD` | Senha do usuário usado para ingressar a VM no domínio (`adcli join`) |
| `SVC_AAP_INITIAL_PASSWORD` | Senha inicial do usuário de serviço `svc_aap` criado na VM (role `create_user`, opcional) |

## Outras variáveis

- `hostname`: nome da VM a criar
- `vm_folder`, `vm_template`, `vm_datastore`: parâmetros de criação no vCenter

## Uso

```bash
ansible-playbook homelab.yaml -e "hostname=meuservidor01"
```

## Estrutura

```text
homelab.yaml                       # Playbook principal (2 plays: criação + configuração)
group_vars/all.yaml                # Variáveis globais (credenciais via lookup env)
roles/
├── vmware/              # Reserva IP no phpIPAM e cria/liga a VM no vCenter
├── inventory/            # Adiciona a VM ao grupo dinâmico mclinux
├── nmcli/                 # Configuração de rede
├── create_user/           # (opcional) Cria usuário de serviço svc_aap com chave SSH da bastion
├── general_config/        # RHSM, SELinux, firewall, banner de login
├── add-packages-default/
├── timedatectl/
└── ad-homelab/             # Ingresso no domínio AD via realmd/sssd
```
