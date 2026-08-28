# ローカル環境から既存システムを Azure にリフトアンドシフトする際の閉域化環境の構築

ローカル環境やオンプレミス環境に存在する既存システムを Azure にリフトアンドシフトする際は、仮想ネットワークで閉域化された環境を構築することが推奨されます。このハンズオンでは、Azure 上における基本的な閉域化環境の構築手順を紹介します。

また、ローカル環境の仮想マシン (VHD、VHDX) を Azure 仮想マシンにリフトアンドシフトする手順についても紹介します。

![システム構成図](./img/AzClosedEnv_system_archtecture.png)

# 概要

このハンズオンでは Azure 仮想ネットワークを使用して閉域化された環境を作成し、同閉域化環境内に Jump Box (踏み台サーバー) を構築します。さらに、ローカル環境の仮想マシンを Azure 仮想マシンにリフトアンドシフトする手順、閉域化環境への Azure Kubernetes Service (AKS) Private Cluster のデプロイ手順についても紹介します。

なお、実際の手順についてはパートごとに分割されたハンズオン資料を使用して実施し、このリポジトリは各ハンズオン資料へのポータルとして使用します。

# 前提条件

この手順を実施する前に、以下の前提条件を満たしている必要があります。

- Azure サブスクリプションが有効であること
- Azure ポータルにアクセスできること
- Azure の管理者権限か共同作成者の権限を持っていること

# この手順で構築される環境

単一の仮想ネットワーク内に、Jump Box (踏み台サーバー) と、ローカル環境からリフトアンドシフトされた Azure 仮想マシン、AKS Private Cluster が構築されます。

仮想ネットワーク内の Azure リソースに対しては Bastion を介した Jump Box のデスクトップを経由してのみアクセスおよび管理が可能となるため、セキュリティの高い閉域化環境を構築することができます。

また、このハンズオンでは触れませんが、この仮想ネットワークに対し、オンプレミス環境から VPN 接続や ExpressRoute 接続を行うことで、オンプレミス環境から Azure 環境への閉域接続を構築することができます。

# 手順

1. [**Azure 仮想ネットワークの作成と Jump Box のデプロイ**](https://github.com/osamum/HowtoMake-Az-JumpBox-Env)

    移行先環境となる Azure 仮想ネットワークを作成し、同仮想ネットワーク内に Jump Box (踏み台サーバー) をデプロイします。
    
    ![Jump Box のシステム構成図](./img/jumpbox_system_archtecture.png)
    
    このハンズオンではすべての Azure リソースは同一リージョンを使用することを前提としているため、仮想ネットワークは 1 つですが、もし複数リージョンにまたがる環境を構築する場合は、各リージョンに仮想ネットワークを作成し、[仮想ネットワークピアリング](https://learn.microsoft.com/azure/virtual-network/virtual-network-peering-overview) を使用して接続し、[仮想ネットワーク リンク](https://learn.microsoft.com/azure/dns/private-dns-virtual-network-links) を使用して、正しく名前解決ができるように構成する必要があります。

2. [**ローカル環境の仮想マシン (VHDX) を Azure に移行するための準備**](https://github.com/osamum/Az_uploadPrep_vhd)

    この手順では、ローカル環境の仮想マシンのファイル (VHD、VHDX) を Azure 仮想マシンにリフトアンドシフトするための準備を行います。

    ハンズオンでは作業に使用する仮想マシンのファイルが存在している前提で記述されていますが、もし学習目的で実施するにあたり、仮想マシンのファイルが存在しない場合は、[\[**Hyper-V クイック作成**\]](https://learn.microsoft.com/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) から Windows 開発環境の仮想マシンを作成し、仮想マシンのファイルを取得することができます。ただし、この仮想マシンに既定で設定されているユーザーのパスワードは公開されていないので、別途、管理者権限を持つローカルアカウントを新規に作成してから作業を行ってください。

    ![Hyper-V クイック作成](./img/Hyper-V-quick.png)

3. [**VHD ファイルの Azure へのアップロードと、それを使用した仮想ネットワーク内の閉域環境への仮想マシンの作成**](https://github.com/osamum/Howto-upload-VHD-to-Azure-and-create-VM-)

    手順 2 で準備した VHD ファイルを Azure にアップロードし、同 VHD ファイルを使用して仮想ネットワーク内に仮想マシンを作成します。デプロイされた仮想マシンへのアクセスは、Jump Box のデスクトップから RDP を使用して行います。

    ![VHD ファイルの Azure へのアップロード](img/VM_from_vhd_system_archtecture.png)

4. [**Azure の仮想ネットワークで閉域化された環境に Azure Kubernetes Service (AKS) を構築する**](https://github.com/osamum/HowtoDeploy_AKS_PrivateCluster)

    仮想ネットワーク内に AKS Private Cluster を構築します。AKS Private Cluster は、仮想ネットワーク内のプライベート IP アドレスを使用して AKS クラスターの API サーバーにアクセスするため、インターネット経由でのアクセスはできません。AKS Private Cluster の管理は、Jump Box のデスクトップから行います。

    ただし、クラスターからのインターネット接続は可能で、このハンズオンではアプリケーションは GitHub Container Registry (GHCR) に格納されている AKS デモアプリケーションのコンテナイメージを使用してデプロイします。

    ![閉域環境に AKS をデプロイ](img/Private_AKS_SystemArchtecture.png)

    もし、コンテナーレジストリも閉域化された環境内に構築したい場合は、別途 [Azure Container Registry (ACR) をデプロイ](https://learn.microsoft.com/azure/container-registry/container-registry-get-started-portal)し、[プライベートリンク](https://learn.microsoft.com/azure/container-registry/container-registry-private-endpoints) を作成して閉域化された環境内からアクセスできるように構成します。その後、ACR 側のネットワーク設定でパブリックネットワークからのアクセスを無効化すれば、閉域化された環境内でのみコンテナイメージを取得できるようになります。

