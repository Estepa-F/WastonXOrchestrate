# Tools

Comment creer les tools

## 📦 Prérequis

- [Python 3.12+](https://www.python.org/)
- [watsonx Agent Development Kit (ADK)](https://developer.watson-orchestrate.ibm.com)

---

## 📦 Imports

- tool: décorateur pour transformer une fonction Python en "outil utilisable" dans un agent Watsonx Orchestrate.

```python
# Tool import
from ibm_watsonx_orchestrate.agent_builder.tools import tool, ToolPermission

#autres imports
import requests #pour envoyer des requêtes HTTP
## ect...

```

---

## 📦 Déclaration de l'outil

- Donne un nom, une description et un niveau de permission à l’outil pour qu’il soit reconnu et utilisé dans un agent Watsonx Orchestrate.

```python
@tool(
    name="list_github_files",
    description="Liste tous les fichiers d'un dépôt GitHub via l'API GitHub.",
    permission=ToolPermission.ADMIN
)

```

- Laisse "ADMIN" en permission pour ne pas avoir de soucis.
- le name va servir à relier le tool à votre agent.

---

## 🧠 Fonction

- Le nom de votre fonction
- les arguments de votre fonction
- l'annotation de type de retour ("-> list" : la fonction retourne une liste)

```python
def list_github_files(owner: str, repo: str, branch: str = "main", token: str = "") -> list:

```

- une description de votre fonction

```python
    """
    Récupère récursivement tous les fichiers d'un dépôt GitHub public ou privé.
    
    Args:
        owner (str): Le nom du propriétaire du dépôt GitHub.
        repo (str): Le nom du dépôt.
        branch (str): La branche (par défaut 'main').
        token (str): Ton token GitHub personnel (facultatif si public).
        
    Returns:
        list: Liste des chemins de fichiers dans le dépôt.
    """
```

- Le code de votre fonction

```python

    headers = {}
    if token:
        headers["Authorization"] = f"token {token}"

    url = f"https://api.github.com/repos/{owner}/{repo}/git/trees/{branch}?recursive=1"
    response = requests.get(url, headers=headers)

    if response.status_code != 200:
        return [f"Erreur {response.status_code} : {response.text}"]

    tree = response.json().get("tree", [])
    files = [item["path"] for item in tree if item["type"] == "blob"]
    
```

- Le Résultat

```python

return files # Le type de retour doit être le même que dans la définition de la fonction
    
```

---

## Exemple complet

```python

from ibm_watsonx_orchestrate.agent_builder.tools import tool, ToolPermission
import requests

@tool(
    name="list_github_files",
    description="Liste tous les fichiers d'un dépôt GitHub via l'API GitHub.",
    permission=ToolPermission.ADMIN
)
def list_github_files(owner: str, repo: str, branch: str = "main", token: str = "") -> list:
    """
    Récupère récursivement tous les fichiers d'un dépôt GitHub public ou privé.
    
    Args:
        owner (str): Le nom du propriétaire du dépôt GitHub.
        repo (str): Le nom du dépôt.
        branch (str): La branche (par défaut 'main').
        token (str): Ton token GitHub personnel (facultatif si public).
        
    Returns:
        list: Liste des chemins de fichiers dans le dépôt.
    """
    headers = {}
    if token:
        headers["Authorization"] = f"token {token}"

    url = f"https://api.github.com/repos/{owner}/{repo}/git/trees/{branch}?recursive=1"
    response = requests.get(url, headers=headers)

    if response.status_code != 200:
        return [f"Erreur {response.status_code} : {response.text}"]

    tree = response.json().get("tree", [])
    files = [item["path"] for item in tree if item["type"] == "blob"]
    
    return files
    
```
