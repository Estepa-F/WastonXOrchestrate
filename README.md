# WastonX Orchestrate

Git pour utiliser WatsonX Orchestrate Agentique

## 📦 Prérequis

- [Python 3.12+](https://www.python.org/)
- [watsonx Agent Development Kit (ADK)](https://developer.watson-orchestrate.ibm.com)
- Un compte IBM Cloud avec accès à watsonx Orchestrate
- Un compte IBM Cloud avec accès à watsonx AI

---

## Installation

Avant de pouvoir importer et exécuter tes agents et tes tools dans watsonx Orchestrate, il faut configurer ton environnement de développement local avec les variables nécessaires à l'exécution du runtime en mode développeur.

### 📄 Étape 1 : Créer un fichier `.env`

À la racine de ton projet, crée un fichier nommé `.env` et colle-y le contenu suivant :

```env
WO_DEVELOPER_EDITION_SOURCE=internal
DOCKER_IAM_KEY="votre-clé-Docker-IAM"
OPENSOURCE_REGISTRY_PROXY=us.icr.io/watson-orchestrate-private
WO_ENTITLEMENT_KEY="votre-clé-d'entitlement-IBM"
WATSONX_APIKEY="votre-API-key-Watsonx"
WATSONX_SPACE_ID="votre-ID-d'espace-Watsonx"
```

### 🔑 Où trouver ces clés ?

#### 🔹 `DOCKER_IAM_KEY`

Clé d’authentification pour télécharger les images nécessaires au runtime local.

- Va sur : [https://github.ibm.com/WatsonOrchestrate/wxo-clients/issues/new/choose](https://github.ibm.com/WatsonOrchestrate/wxo-clients/issues/new/choose)
- Clique sur **"Get started"**pour faire une demande. La clée sera envoyé par email après validation des équipes IBM.

![Key Request](docs/img/KeyRequest.png)

- Copie la clé et colle-la dans `DOCKER_IAM_KEY`

---

#### 🔹 `WO_ENTITLEMENT_KEY`

Clé donnant accès aux images privées d’IBM dans le container registry.

- Va sur : [https://myibm.ibm.com/products-services/containerlibrary](https://myibm.ibm.com/products-services/containerlibrary)
- Si ta clé n'éxiste pas Clique sur **"Add new key"**
- Copie la clé et colle-la dans `WO_ENTITLEMENT_KEY`

![Entitlement Key](docs/img/EntitlementKey.png)

---

#### 🔹 `WATSONX_APIKEY`

Clé d’accès aux modèles d’IA et services dans Watsonx.ai.

- Va sur ta réservation techzone (Par exemple: [Techzone environnement](https://techzone.ibm.com/my/reservations/create/64e6866b41bf2a0017d986ad))
- En bas de ta page récupère dans la partie **Reservation Details** la IBM Cloud API Key

![APIKey](docs/img/IBMAPIKEY.png)

- Colle-la dans `WATSONX_APIKEY`

---

#### 🔹 `WATSONX_SPACE_ID`

Identifiant unique de l’espace de deployment Watsonx.AI.

- Va sur ton environnement WatsonX.AI et clique sur "View all deployment spaces"

![View deployments](docs/img/ViewDeployments.png)

- Clique sur **"New deployment space"**

![New deployment](docs/img/newDeployment.png)

- Donne un nom et un description puis clique sur **Create**

![Create deployment](docs/img/createDeployment.png)

- Clique sur l'onglet **Manage** et récupère le **Space GUID**

![Manage deployment](docs/img/manageDeployment.png)

- Colle-le dans `WATSONX_SPACE_ID`

## 🧠 Créer un Tool pour IBM watsonx Orchestrate

Ce dépôt explique comment créer des **tools personnalisés** pour [IBM watsonx Orchestrate](https://developer.watson-orchestrate.ibm.com). Un *tool* est une fonction Python décorée qui permet à un agent d'exécuter une action automatisée, comme interagir avec une API externe ou transformer des données.

### 🚀 Exemples de Tools

#### 🔹 1. `list_github_files`: Lister les fichiers d’un dépôt GitHub

```python
from ibm_watsonx_orchestrate.agent_builder.tools import tool, ToolPermission
import requests

@tool(
    name="list_github_files",
    description="Liste tous les fichiers d'un dépôt GitHub via l'API GitHub.",
    permission=ToolPermission.ADMIN
)
def list_github_files(owner: str, repo: str, branch: str = "main", token: str = "") -> list:
    headers = {}
    if token:
        headers["Authorization"] = f"token {token}"

    url = f"https://api.github.com/repos/{owner}/{repo}/git/trees/{branch}?recursive=1"
    response = requests.get(url, headers=headers)

    if response.status_code != 200:
        return [f"Erreur {response.status_code} : {response.text}"]

    tree = response.json().get("tree", [])
    return [item["path"] for item in tree if item["type"] == "blob"]
