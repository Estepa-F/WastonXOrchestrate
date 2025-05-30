# WastonX Orchestrate

Git pour utiliser WatsonX Orchestrate Agentique

## Installation

## 🧠 Créer un Tool pour IBM watsonx Orchestrate

Ce dépôt explique comment créer des **tools personnalisés** pour [IBM watsonx Orchestrate](https://developer.watson-orchestrate.ibm.com). Un *tool* est une fonction Python décorée qui permet à un agent d'exécuter une action automatisée, comme interagir avec une API externe ou transformer des données.

### 📦 Prérequis

- [Python 3.8+](https://www.python.org/)
- [watsonx Agent Development Kit (ADK)](https://developer.watson-orchestrate.ibm.com)
- Un compte IBM Cloud avec accès à watsonx Orchestrate
- Facultatif : un token GitHub si vous interagissez avec des dépôts privés

---

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
