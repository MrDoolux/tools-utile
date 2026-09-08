---
title: Filigrane JPEG/PNG avec ImageMagick
tags:
  - imagemagick
  - powershell
  - bash
  - debian
  - redhat
  - filigrane
  - howto
---

# Filigrane avec ImageMagick

Tamponner des JPEG/PNG en diagonale, sans écraser les originaux.

Même outil partout. Ce qui change : l’install, le terminal, la police, la boucle.

| Système | Terminal | Binaire | Police par défaut |
|---|---|---|---|
| Windows | PowerShell | `magick` (IM7) | `Arial-Bold` |
| Debian / Ubuntu | bash | `magick` ou `convert` | `DejaVu-Sans-Bold` |
| Fedora / RHEL / Rocky / Alma | bash | `magick` ou `convert` | `DejaVu-Sans-Bold` |

Sur Linux, si `magick` n’existe pas, remplace `magick` par `convert` dans toutes les commandes.

---

## 1. Installer

### Windows (PowerShell)

```powershell
winget install ImageMagick.Q16-HDRI
```

Fermer le terminal, le rouvrir.

Installateur manuel (cocher **Add to PATH**) :
https://imagemagick.org/script/download.php#windows

### Debian / Ubuntu / Mint

```bash
sudo apt update
sudo apt install -y imagemagick fonts-dejavu-core
```

### Fedora

```bash
sudo dnf install -y ImageMagick dejavu-sans-fonts
```

### RHEL / Rocky / Alma / CentOS Stream

```bash
sudo dnf install -y ImageMagick dejavu-sans-fonts
```

Ancien CentOS 7 (yum) :

```bash
sudo yum install -y ImageMagick
```

### Vérifier (tous)

```bash
magick -version
```

Si ça échoue :

```bash
convert -version
```

Tu dois voir `ImageMagick` et un numéro de version. Note quel binaire marche (`magick` ou `convert`) et utilise celui-là partout.

Polices dispo :

```bash
magick -list font | grep -iE "dejavu|liberation|arial|free"
```

---

## 2. Aller dans TON dossier

### Windows (PowerShell)

```powershell
cd "E:\TON_DOSSIER"
```

Ou : tape `cd ` (espace), glisse le dossier dans la fenêtre, Entrée.

```powershell
Get-ChildItem *.png,*.jpg
```

### Linux (Debian / Red Hat)

```bash
cd /chemin/vers/ton/dossier
ls *.jpg *.png
```

Liste vide = mauvais dossier.

---

## 3. Tester UN fichier

Remplace `Image.jpg` par le vrai nom. Espaces et parenthèses : entre guillemets.

### Windows

```powershell
magick "Image.jpg" -gravity center -font Arial-Bold -pointsize 250 -fill none -stroke "rgba(200,20,20,0.40)" -strokewidth 4 -annotate -45x-45+0+0 "COPIE UTILISATION WEB" "filigrane_Image.jpg"
```

### Linux (Debian / Red Hat)

```bash
magick "Image.jpg" -gravity center -font DejaVu-Sans-Bold -pointsize 250 -fill none -stroke "rgba(200,20,20,0.40)" -strokewidth 4 -annotate -45x-45+0+0 "COPIE UTILISATION WEB" "filigrane_Image.jpg"
```

Si `magick` est inconnu :

```bash
convert "Image.jpg" -gravity center -font DejaVu-Sans-Bold -pointsize 250 -fill none -stroke "rgba(200,20,20,0.40)" -strokewidth 4 -annotate -45x-45+0+0 "COPIE UTILISATION WEB" "filigrane_Image.jpg"
```

Ça crée `filigrane_Image.jpg` à côté. L’original ne bouge pas. Relancer la ligne écrase seulement la copie.

---

## 4. Traiter tout le dossier

Ignore les fichiers déjà préfixés `filigrane_`.

### Windows

```powershell
Get-ChildItem *.png,*.jpg | Where-Object { $_.Name -notlike "filigrane_*" } | ForEach-Object { magick $_.FullName -gravity center -font Arial-Bold -pointsize 250 -fill none -stroke "rgba(200,20,20,0.40)" -strokewidth 4 -annotate -45x-45+0+0 "COPIE UTILISATION WEB" "filigrane_$($_.Name)" }
```

### Linux (Debian / Red Hat)

```bash
for f in *.jpg *.png *.JPG *.PNG; do
  [ -f "$f" ] || continue
  case "$f" in filigrane_*) continue ;; esac
  magick "$f" -gravity center -font DejaVu-Sans-Bold -pointsize 250 -fill none -stroke "rgba(200,20,20,0.40)" -strokewidth 4 -annotate -45x-45+0+0 "COPIE UTILISATION WEB" "filigrane_$f"
done
```

Même boucle avec `convert` si tu n’as pas `magick` : remplace uniquement le mot `magick` par `convert`.

Tu envoies uniquement les `filigrane_…`.

---

## 5. Réglages (identiques partout)

| Tu veux | Tu changes |
|---|---|
| Texte plus grand / toute la diagonale | `-pointsize 250` → `280` ou `300` |
| Texte plus petit | `-pointsize 250` → `180` |
| L’autre diagonale | `-45x-45+0+0` → `45x45+0+0` |
| Rouge plus fort | `0.40` → `0.55` |
| Rouge plus faded | `0.40` → `0.25` |
| Contour plus épais | `-strokewidth 4` → `6` |
| Lettres pleines, plus de creux | `-fill none` → `-fill "rgba(200,20,20,0.20)"` |
| Autre texte | `"COPIE UTILISATION WEB"` |

### Taille auto — Windows

```powershell
$w = [int](magick identify -format "%w" "Image.jpg")
$ps = [math]::Floor($w / 11)
magick "Image.jpg" -gravity center -font Arial-Bold -pointsize $ps -fill none -stroke "rgba(200,20,20,0.40)" -strokewidth 4 -annotate -45x-45+0+0 "COPIE UTILISATION WEB" "filigrane_Image.jpg"
```

### Taille auto — Linux

```bash
w=$(identify -format "%w" "Image.jpg")
ps=$((w / 11))
magick "Image.jpg" -gravity center -font DejaVu-Sans-Bold -pointsize "$ps" -fill none -stroke "rgba(200,20,20,0.40)" -strokewidth 4 -annotate -45x-45+0+0 "COPIE UTILISATION WEB" "filigrane_Image.jpg"
```

Sur IM7, `identify` peut s’écrire `magick identify`.

---

## 6. Textes utiles

- Web public : `COPIE UTILISATION WEB`
- Court : `COPIE` / `COPIE WEB`
- Neutre : `DOCUMENT NON ORIGINAL`

## 7. Quand l’utiliser

- **Web public** (site, réseau) : filigrane oui. Masquer numéro + QR du titre.
- **Recruteur / dossier de candidature** : pas de filigrane, pas de QR caché. Copie lisible. Casier, permis, évaluations : seulement si on les demande.

## 8. Erreurs fréquentes

| Message / symptôme | Cause | Quoi faire |
|---|---|---|
| `magick` n’est pas reconnu (Windows) | Pas installé, ou terminal pas relancé | Réinstaller / fermer-rouvrir PowerShell |
| `magick` n’est pas reconnu (Linux) | Paquet IM6, binaire = `convert` | Utiliser `convert` |
| `permission denied` / policy (Linux) | Policy ImageMagick trop stricte | Rare sur JPEG/PNG. Ne pas toucher la policy pour du PDF ici |
| Fichier introuvable | Mauvais dossier | `ls *.jpg` ou `Get-ChildItem *.jpg` |
| Texte à l’envers | Mauvais signe de rotation | `-45` ou `45` |
| Deux filigranes superposés | Relancé sur `filigrane_*.jpg` | Toujours partir de l’original |
| Police en pavés | Police absente | `magick -list font` puis changer `-font` |
| `*.png: no matches found` (Linux zsh) | Pas de PNG dans le dossier | Normal. La boucle avec `[ -f "$f" ]` ignore ça |

## 9. Rappels

- Toujours écrire vers `filigrane_…`, jamais sur l’original.
- Un filigrane visible décourage. Ça n’empêche pas un recadrage.
- Une image, sans terminal : https://www.iloveimg.com/watermark-image
