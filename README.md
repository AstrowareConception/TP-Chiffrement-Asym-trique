# 🛡️ TP – Chiffrement Asymétrique et Coffre-Fort Numérique (Python)

## 🎯 Objectifs pédagogiques

À la fin de ce TP, vous serez capables de :

* Comprendre les différences entre **chiffrement symétrique** et **chiffrement asymétrique**.
* Générer des **paires de clés RSA (publique/privée)**.
* Utiliser une clé publique pour **chiffrer un message ou fichier**.
* Utiliser une clé privée pour **déchiffrer** ce contenu.
* Mettre en place un **système d’échange sécurisé** entre vous, simulant un **coffre-fort numérique**.

Durée : **2h**
Niveau : BTS SIO 2ᵉ année

---

## 1. 📚 Partie Théorique

### 1.1. Pourquoi chiffrer ?

Aujourd’hui, la sécurité des données est un enjeu majeur :

* protéger les **données personnelles**,
* sécuriser les **communications (emails, messageries, web)**,
* garantir la **confidentialité et l’intégrité** des échanges.

⚠️ Exemple concret : sans chiffrement, un mot de passe envoyé sur Internet pourrait être intercepté par n’importe qui.

---

### 1.2. Chiffrement symétrique vs asymétrique

* **Chiffrement symétrique**

  * Une seule clé sert à **chiffrer** et à **déchiffrer**.
  * Exemple : AES.
  * Problème : comment échanger la clé de façon sécurisée ?

* **Chiffrement asymétrique**

  * Deux clés différentes :

    * **Clé publique** : peut être diffusée à tous.
    * **Clé privée** : doit rester secrète.
  * On chiffre avec la clé publique → seul celui qui a la clé privée correspondante peut déchiffrer.
  * Exemple : RSA.
  * Utilisé dans HTTPS, SSH, coffres-forts numériques, etc.

---

### 1.3. Exemple d’usage

* Alice veut envoyer un message secret à Bob.
* Bob publie sa **clé publique**.
* Alice chiffre le message avec la clé publique de Bob.
* Seule la **clé privée de Bob** permet de le déchiffrer.

Résultat : même si un attaquant intercepte le message, il ne pourra pas le lire sans la clé privée.

---

## 2. 🖥️ Partie Pratique (Python)

Nous allons utiliser la librairie **cryptography** (déjà incluse dans beaucoup d’environnements Python, sinon installation avec `pip install cryptography`).

### 2.1. Génération d’une paire de clés RSA

👉 Objectif : chaque étudiant doit générer sa **clé publique** et sa **clé privée**.

```python
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization

# Génération des clés
private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048
)
public_key = private_key.public_key()

# Sauvegarde de la clé privée
with open("ma_cle_privee.pem", "wb") as f:
    f.write(private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.TraditionalOpenSSL,
        encryption_algorithm=serialization.NoEncryption()
    ))

# Sauvegarde de la clé publique
with open("ma_cle_publique.pem", "wb") as f:
    f.write(public_key.public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo
    ))

print("Clés générées et sauvegardées dans des fichiers PEM !")
```

👉 À ce stade, vous devez avoir deux fichiers dans votre dossier :

* `ma_cle_privee.pem`
* `ma_cle_publique.pem`

---

### 2.2. Chiffrement d’un message avec la clé publique

👉 Objectif : chiffrer un message pour un camarade.

```python
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives import hashes

# Chargement de la clé publique depuis un fichier
with open("ma_cle_publique.pem", "rb") as f:
    public_key = serialization.load_pem_public_key(f.read())

# Message secret à envoyer
message = "Bravo ! Tu as réussi à déchiffrer ce message 🔐"

# Chiffrement
ciphertext = public_key.encrypt(
    message.encode(),
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None
    )
)

# Sauvegarde du message chiffré dans un fichier
with open("message_chiffre.bin", "wb") as f:
    f.write(ciphertext)

print("Message chiffré et sauvegardé dans message_chiffre.bin")
```

---

### 2.3. Déchiffrement avec la clé privée

👉 Objectif : récupérer le message avec la clé privée.

```python
# Chargement de la clé privée
with open("ma_cle_privee.pem", "rb") as f:
    private_key = serialization.load_pem_private_key(f.read(), password=None)

# Chargement du message chiffré
with open("message_chiffre.bin", "rb") as f:
    ciphertext = f.read()

# Déchiffrement
plaintext = private_key.decrypt(
    ciphertext,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None
    )
)

print("Message déchiffré :", plaintext.decode())
```

---

## 3. 🎲 Activité Ludique : Agents Secrets

### Étape 1 : Génération des clés

* Chaque étudiant génère sa paire de clés RSA (`ma_cle_privee.pem` et `ma_cle_publique.pem`).
* La clé privée reste secrète.
* La clé publique est partagée (par USB, Moodle, Discord, etc.).

### Étape 2 : Envoi d’un message secret

* Vous choisissez un camarade.
* Vous récupérez sa **clé publique**.
* Vous chiffrez un message secret et vous lui envoyez le fichier `message_chiffre.bin`.

### Étape 3 : Déchiffrement

* Vous utilisez **votre clé privée** pour lire le message reçu.
* Vous prouvez que vous êtes le seul capable de déchiffrer.

### Étape 4 : Jeu de rôle

* Imaginez que vous êtes des **agents secrets**.
* Votre mission est de transmettre une information ultra-confidentielle.
* Seul le bon destinataire pourra la lire.
* Si un autre étudiant intercepte le message → impossible de déchiffrer !

---

## 4. 🚀 Pour aller plus loin (si temps)

* Chiffrer **des fichiers** au lieu de simples messages.
* Ajouter une **signature numérique** (l’expéditeur signe avec sa clé privée → le destinataire vérifie avec la clé publique).
* Mélanger **asymétrique** et **symétrique** (comme dans HTTPS) :

  * RSA pour échanger une clé secrète.
  * AES pour chiffrer un gros fichier.

---

## 5. ✅ Bilan attendu

À la fin du TP, vous devez être capables de :

* Expliquer le rôle des clés publique/privée.
* Générer vos propres clés.
* Envoyer un message secret qu’un camarade peut déchiffrer.
* Constater qu’aucun autre étudiant ne peut le lire.

👉 Vous venez de construire la **brique de base d’un coffre-fort numérique** !
