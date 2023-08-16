## lab-01
otus | first terraform script

### Домашнее задание
первый терраформ скрипт

#### Цель:
реализовать первый терраформ скрипт.

#### Описание/Пошаговая инструкция выполнения домашнего задания:
##### Необходимо:
1. реализовать терраформ для разворачивания одной виртуалки в yandex-cloud
2. запровиженить nginx с помощью ansible

#### Формат сдачи
- репозиторий с терраформ манифестами
- README файл

#### Критерии оценки:
преподаватель с помощью terraform apply должен получить развернутый стенд

### Выполнение домашнего задания

#### Создание стенда

Получаем OAUTH токен:
```
https://cloud.yandex.ru/docs/iam/concepts/authorization/oauth-token
```
Настраиваем аутентификации в консоли:
```
export YC_TOKEN=$(yc iam create-token)
export TF_VAR_yc_token=$YC_TOKEN
```

Создадим директорий lab-01 и перейдём в него:
```
mkdir ./lab-01 && cd ./lab-01/
```
Создадим необходимые файлы:
provider.tf:
```
locals {
  cloud_id           = "b1gk2uh1jv4i27fikj4f"
  folder_id          = "b1g5h8d28qvg63eps3ms" #otus-lab
  zone               = "ru-central1-b"
}

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  cloud_id  = local.cloud_id
  folder_id = local.folder_id
}
```

variables.tf:
```
variable "vpc_name" {
  type = string
  description = "VPC name"
}

variable "zone" {
  type = string
  default = "ru-central1-b"
  description = "zone"
}

variable "subnet_name" {
  type = string
  description = "subnet name"
}

variable "subnet_cidrs" {
  type = list(string)
  description = "CIDRs"
}

## VM parameters
variable "vm_name" {
  description = "VM name"
  type        = string
}

variable "cpu" {
  description = "VM CPU count"
  default     = 2
  type        = number
}

variable "memory" {
  description = "VM RAM size"
  default     = 4
  type        = number
}

variable "core_fraction" {
  description = "Core fraction, default 100%"
  default     = 100
  type        = number
}

variable "disk" {
  description = "VM Disk size"
  default     = 10
  type        = number
}

variable "image_id" {
  description = "Default image ID Debian 11"
  default     = "fd83u9thmahrv9lgedrk" # debian-11-v20230814
  type        = string
}

variable "nat" {
  type    = bool
  default = true
}

variable "platform_id" {
  type    = string
  default = "standard-v3"
}

variable "internal_ip_address" {
  type    = string
  default = null
}

variable "nat_ip_address" {
  type    = string
  default = null
}

variable "disk_type" {
  description = "Disk type"
  type        = string
  default     = "network-ssd"
}

variable "ssh_key" {
  type        = string
  description = "cloud-config ssh key"
  default = ""
}
```

main.tf:
```
locals {
  ssh_key = "user:${file("~/.ssh/id_rsa.pub")}"
}

resource "yandex_vpc_network" "vpc" {
  # folder_id = var.folder_id
  name = var.vpc_name
}

resource "yandex_vpc_subnet" "subnet" {
  # folder_id = var.folder_id
  v4_cidr_blocks = var.subnet_cidrs
  zone           = var.zone
  name           = var.subnet_name
  network_id = yandex_vpc_network.vpc.id
}

resource "yandex_compute_instance" "instance" {
  name        = var.vm_name
  hostname    = var.vm_name
  platform_id = var.platform_id
  zone        = var.zone
  # folder_id   = var.folder_id
  resources {
    cores         = var.cpu
    memory        = var.memory
    core_fraction = var.core_fraction
  }

  boot_disk {
    initialize_params {
      image_id = var.image_id
      size     = var.disk
      type     = var.disk_type
    }
  }

  network_interface {
    subnet_id          = yandex_vpc_subnet.subnet.id
    nat                = var.nat
    ip_address         = var.internal_ip_address
    nat_ip_address     = var.nat_ip_address
  }

  metadata = {
    ssh-keys           = local.ssh_key
  }
}
```
input.auto.tfvars:
```
vpc_name = "demo_vpc"
subnet_name = "demo_subnet"
subnet_cidrs = [ "10.10.10.0/24" ]

vm_name = "demo-vm"
```
Для инициализации проекта запустим команду:
```
terraform init
```
```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of yandex-cloud/yandex...
- Installing yandex-cloud/yandex v0.96.1...
- Installed yandex-cloud/yandex v0.96.1 (unauthenticated)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

╷
│ Warning: Incomplete lock file information for providers
│ 
│ Due to your customized provider installation methods, Terraform was forced to calculate lock file checksums locally for the
│ following providers:
│   - yandex-cloud/yandex
│ 
│ The current .terraform.lock.hcl file only includes checksums for linux_amd64, so Terraform running on another platform will fail to
│ install these providers.
│ 
│ To calculate additional checksums for another platform, run:
│   terraform providers lock -platform=linux_amd64
│ (where linux_amd64 is the platform to generate)
╵

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
Проверим на корректность синтаксиса написания:
```
terraform validate
```
```
Success! The configuration is valid.
```

Следующей командой увидим план предстоящего выполнения проекта:
```
terraform plan
```
Получим следующее:
```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # yandex_compute_instance.instance will be created
  + resource "yandex_compute_instance" "instance" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = "demo-vm"
      + id                        = (known after apply)
      + metadata                  = {
          + "ssh-keys" = <<-EOT
                user:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDOhoQz97bl8n/F/QyM1OiqgrdA4ClCRVKBAZGDHtAF31UlrM+OQ6d33yhlDUMfMyzubeXGt0yeiRaPYEDPe41Pa3EkDd8S2tRtgMfu2iSO6QeYiwQRhPmbJQNLDfLD0GI0UOVw76KjxxA6iTNon5wvgeZfEzcNHP/OluYrtdPYuYmSrSMSO9xaXyWvkZgwDsNQPJYf2PhxPgCkVwNyBA64NVEKsUqpC6RWpxC2GTknq8cgBp5/lJr+Z7b6bTnP39tUSd4ansaXGJ960B+lS/AuZA7TpGFaq1NWycPBBt24DAGvWMWcTfEuByT+uXLHYd7J018lfSL2HL9TE3wEmnFEnkpTxqU9Jih4ToYcE6533fy2+TjIsGwszUb/x4TVRgrUFmtjRyYYc3rKFQrh9dncQBm6cAbB35bTtBysnYAspDQdGtcD4ULOBn30bWPxV1wj8lXjTxrG3K3qEK5TVPB01ITdnp9Kd7gaouMg44t6sOIlKybBUtn88/D1hhtiWRTXdZX3qheHvubrF76AkAJ0VolIidGTX0++TrR5qqjEkrsSAa4nNVmr47NZjDYRxW/twCK71eRm+cYwYDnnAXVf4pmjANebCrQ+zEvu/dPil0XVCUPHXY2I2O/mjmpDJkz16f6dlbY1jo/Vhh69IMOJ0KsWBQwsv6oMHZNKYHmNiQ== user@redos.localdomain
            EOT
        }
      + name                      = "demo-vm"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v3"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = "ru-central1-b"

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd83u9thmahrv9lgedrk"
              + name        = (known after apply)
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-ssd"
            }
        }

      + metadata_options {
          + aws_v1_http_endpoint = (known after apply)
          + aws_v1_http_token    = (known after apply)
          + gce_http_endpoint    = (known after apply)
          + gce_http_token       = (known after apply)
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy {
          + host_affinity_rules = (known after apply)
          + placement_group_id  = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 4
        }

      + scheduling_policy {
          + preemptible = (known after apply)
        }
    }

  # yandex_vpc_network.vpc will be created
  + resource "yandex_vpc_network" "vpc" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "demo_vpc"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.subnet will be created
  + resource "yandex_vpc_subnet" "subnet" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "demo_subnet"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.10.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-b"
    }

Plan: 3 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run
"terraform apply" now.
```
Теперь выполним проект с помощью следующей команды:
```
terraform apply
```
```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # yandex_compute_instance.instance will be created
  + resource "yandex_compute_instance" "instance" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = "demo-vm"
      + id                        = (known after apply)
      + metadata                  = {
          + "ssh-keys" = <<-EOT
                user:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDOhoQz97bl8n/F/QyM1OiqgrdA4ClCRVKBAZGDHtAF31UlrM+OQ6d33yhlDUMfMyzubeXGt0yeiRaPYEDPe41Pa3EkDd8S2tRtgMfu2iSO6QeYiwQRhPmbJQNLDfLD0GI0UOVw76KjxxA6iTNon5wvgeZfEzcNHP/OluYrtdPYuYmSrSMSO9xaXyWvkZgwDsNQPJYf2PhxPgCkVwNyBA64NVEKsUqpC6RWpxC2GTknq8cgBp5/lJr+Z7b6bTnP39tUSd4ansaXGJ960B+lS/AuZA7TpGFaq1NWycPBBt24DAGvWMWcTfEuByT+uXLHYd7J018lfSL2HL9TE3wEmnFEnkpTxqU9Jih4ToYcE6533fy2+TjIsGwszUb/x4TVRgrUFmtjRyYYc3rKFQrh9dncQBm6cAbB35bTtBysnYAspDQdGtcD4ULOBn30bWPxV1wj8lXjTxrG3K3qEK5TVPB01ITdnp9Kd7gaouMg44t6sOIlKybBUtn88/D1hhtiWRTXdZX3qheHvubrF76AkAJ0VolIidGTX0++TrR5qqjEkrsSAa4nNVmr47NZjDYRxW/twCK71eRm+cYwYDnnAXVf4pmjANebCrQ+zEvu/dPil0XVCUPHXY2I2O/mjmpDJkz16f6dlbY1jo/Vhh69IMOJ0KsWBQwsv6oMHZNKYHmNiQ== user@redos.localdomain
            EOT
        }
      + name                      = "demo-vm"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v3"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = "ru-central1-b"

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd83u9thmahrv9lgedrk"
              + name        = (known after apply)
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-ssd"
            }
        }

      + metadata_options {
          + aws_v1_http_endpoint = (known after apply)
          + aws_v1_http_token    = (known after apply)
          + gce_http_endpoint    = (known after apply)
          + gce_http_token       = (known after apply)
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy {
          + host_affinity_rules = (known after apply)
          + placement_group_id  = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 4
        }

      + scheduling_policy {
          + preemptible = (known after apply)
        }
    }

  # yandex_vpc_network.vpc will be created
  + resource "yandex_vpc_network" "vpc" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "demo_vpc"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.subnet will be created
  + resource "yandex_vpc_subnet" "subnet" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "demo_subnet"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.10.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-b"
    }

Plan: 3 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 
```
Как видим, команда сделала такой же вывод как и terraform plan, затем набираем 'yes' и нажимаем enter:
```
  Enter a value: yes

yandex_vpc_network.vpc: Creating...
yandex_vpc_network.vpc: Creation complete after 1s [id=enpp6f08k9jo6vum6qre]
yandex_vpc_subnet.subnet: Creating...
yandex_vpc_subnet.subnet: Creation complete after 1s [id=e2lml1vmag3fb7kuf2tg]
yandex_compute_instance.instance: Creating...
yandex_compute_instance.instance: Still creating... [10s elapsed]
yandex_compute_instance.instance: Still creating... [20s elapsed]
yandex_compute_instance.instance: Still creating... [30s elapsed]
yandex_compute_instance.instance: Still creating... [40s elapsed]
yandex_compute_instance.instance: Creation complete after 43s [id=epdloutbt6i24ejhius7]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```
Как видим, было создано три ресурса: внешняя и внутренняя сети и сама виртаульная машина.

<img src="screens/screen-001.png" alt="screen-001.png" />

Удалим полностью все созданные ресурсы с помощью команды:
```
terraform destroy
```