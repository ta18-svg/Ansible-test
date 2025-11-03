# Ansible Playbook: Nginx 自動構築・起動設定

このプロジェクトは、**Ansible** を使用して複数の Amazon Linux サーバー上に  
**Nginx を自動インストール・起動・有効化・状態確認** する構成管理Playbookです。

## 概要

本Playbookは、インフラ構成の自動化（Infrastructure as Code）を目的として作成しました。  
対象サーバー群（`devservers`, `teetservers`）に対し、Nginxの導入から動作確認までを行いました。

## システム構成

**構成管理ツール**: Ansible<br>
**対象OS**:　Amazon Linux 2023<br>
**構成ファイル**:　install_nginx.yaml, inventory.txt, ansible.cfg, hosts<br>
**対象ホストグループ**: devservers, testservers<br>
**主要タスク**:  Nginxの導入・起動・有効化・稼働確認<br>

## システム構成図（Master 1台 → Targets 4台）

```mermaid
graph LR
  subgraph Controller["Ansible Master (EC2: controller-1)"]
    A1[Playbook: install_nginx.yaml]
    A2[Inventory: inventory.txt]
    A3[ansible.cfg (/etc/ansible/)]
    A4[hosts(/etc/ansible/)]
  end

  subgraph Targets["Managed Nodes (Amazon Linux 2023)"]
    T1[(ansible-dev-target1: 35.78.72.250)]
    T2[(ansible-dev-target2: 35.77.223.151)]
    T3[(ansible-test-target1: 13.114.42.248)]
    T4[(ansible-test-target2: 54.250.192.204)]
  end

  A1 -->|SSH + Ansible Modules| T1
  A1 -->|SSH + Ansible Modules| T2
  A1 -->|SSH + Ansible Modules| T3
  A1 -->|SSH + Ansible Modules| T4

```

## ディレクトリ構成
### ec2-user のホームディレクトリ
/home/ec2-user/<br>
├── install_nginx.yaml      ← Playbook（Nginx インストール用）<br>
└── inventory.txt           ← インベントリ（対象ホスト定義）<br>
### Ansible のシステム設定ディレクトリ
/etc/ansible/<br>
├── ansible.cfg             ← 全体設定ファイル<br>
└── hosts                   ← デフォルトインベントリ<br>


## 実行手順
### EC2インスタンス（Master 1台  Targets 4台）作成
 マスターマシンへSSH接続できるように設定しマスターマシンへSSH接続

###　システム全体のパッケージを最新状態に更新する
sudo dnf update -y
###　ansible インストール
sudo yum install ansible
###　インストール確認
ansible --version

##　inventoryファイル作成
vim inventory.txt

## 疎通確認
### 鍵を.sshへコピー
sudo cp test.pem ~/.ssh/
### 所有権＆権限変更（鍵は自分だけが読める状態に）
sudo chown ec2-user:ec2-user ~/.ssh/test.pem
chmod 600 ~/.ssh/test.pem
chmod 700 ~/.ssh
### SSHエージェントを起動
ssh-agent bash
### エージェントに鍵を登録
ssh-add ~/.ssh/test.pem
### 認識確認
ssh-add -l
### ansible-dev-target1へpingで疎通確認
ansible ansible-dev-target1 -m ping -i inventory.txt

## 疎通確認エラー発生の対処　
### そもそもデフォルトでansible.cfgがない→自分で作成
ansible-config init --disabled | sudo tee /etc/ansible/ansible.cfg >/dev/null
### /etc/ansible/へ移動 
cd /etc/ansible/ 
### ansible.cfgを編集
;host_key_checking = False    ;←をはずす。　#と思ったら実際は;だった   TrueをFalse変更<br>  
sudo vim ansible.cfg
## hostsファイル作成
sudo vim hosts

## playbookの作成
vim install_nginx.yaml


## Ansible の Playbook 実行　確認。
ansible-playbook install_nginx.yaml -v