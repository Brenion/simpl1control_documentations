## Documentation : Extraction de la filesystem ESP32 sous PlatformIO

### Contexte

Vous souhaitiez extraire le contenu SPIFFS/LittleFS d’un ESP32 via PlatformIO en utilisant le script `download_fs.py`. Plusieurs erreurs se sont succédé avant d’arriver à une solution stable.

---

## 1. Erreurs rencontrées

1. **Exécutable introuvable**
    
    - Erreur : `"mkspiffs_${PIOPLATFORM}_${PIOFRAMEWORK}" n'est pas reconnu ...`
        
    - Cause : Le script cherchait un binaire nommé selon un format générique, alors que PlatformIO installe des exécutables nommés `mkspiffs_espressif32_arduino.exe`, etc.
        
2. **Erreur d’indentation**
    
    - Erreur : `IndentationError: unindent does not match any outer indentation level`
        
    - Cause : Mélange d’espaces et de tabulations dans la définition de la classe `FS_Info`.
        
3. **NameError: self non défini**
    
    - Erreur : `NameError: name 'self' is not defined` lors de la déclaration de `self.tool` hors méthode.
        
    - Cause : Mauvaise indentation du code, `self` étant hors du contexte de l’`__init__`.
        
4. **Partition corrompue**
    
    - Erreur : `lfs error:969: Corrupted dir pair at 0 1`
        
    - Cause : Script appelait `mklittlefs` sur une partition SPIFFS (ou vice-versa).
        
5. **Installation des tools incorrecte**
    
    - Erreur : `Error: Got unexpected extra argument (tool-mkspiffs)`
        
    - Cause : Mauvaise syntaxe de la commande `pio pkg install`.
        

---

## 2. Solutions appliquées

### 2.1 Installation correcte des outils

- Pour SPIFFS :
    
    ```bash
    pio pkg install -t "platformio/tool-mkspiffs"
    ```
    
- Pour LittleFS :
    
    ```bash
    pio pkg install -t "platformio/tool-mklittlefs"
    ```
    
- Vérification : les dossiers contenant `mkspiffs_espressif32_...` et `mklittlefs_espressif32_...` sont désormais présents.
    

### 2.2 Mise à jour du script `download_fs.py`

- **Classe `FS_Info`**
    
    - Détection dynamique du package (`tool-mkspiffs` ou `tool-mklittlefs`) selon `board_build.filesystem`.
        
    - Recherche automatique du fichier exécutable dont le nom commence par `mkspiffs` ou `mklittlefs`.
        
- **Implémentation de `get_extract_cmd`**
    
    - Retourne une liste d’arguments pour `subprocess`, évitant les problèmes de guillemets sous Windows.
        
- **Extraction via `subprocess.check_call(cmd, shell=False)`**
    
    - Appel direct de l’exécutable avec arguments listés.
        

---

## 3. Extrait du code final (`download_fs.py`)

```python
class FS_Info(FSInfo):
    def __init__(self, start, length, page_size, block_size):
        fs = board.get("build.filesystem", "littlefs")
        if fs == "littlefs":
            pkg = "tool-mklittlefs"
            prefix = "mklittlefs"
            fs_type = FSType.LITTLEFS
        else:
            pkg = "tool-mkspiffs"
            prefix = "mkspiffs"
            fs_type = FSType.FATFS

        tool_path = platform.get_package_dir(pkg)
        candidates = [
            f for f in os.listdir(tool_path)
            if f.lower().startswith(prefix) and os.access(os.path.join(tool_path, f), os.X_OK)
        ]
        exe_name = candidates[0]
        self.tool = os.path.join(tool_path, exe_name)
        super().__init__(fs_type, start, length, page_size, block_size)

    def get_extract_cmd(self, input_file, output_dir):
        return [
            self.tool,
            "-b", str(self.block_size),
            "-s", str(self.length),
            "-p", str(self.page_size),
            "--unpack", output_dir,
            input_file
        ]
```

---

## 4. Utilisation finale

1. Dans `platformio.ini`, déterminez le FS :
    
    ```ini
    board_build.filesystem = spiffs  ; ou littlefs
    ```
    
2. Installez l’outil correspondant (`tool-mkspiffs`/`tool-mklittlefs`).
    
3. Supprimez `unpacked_fs` en cas d’extraction précédente.
    
4. Lancez la commande :
    
    ```bash
    pio run -t downloadfs
    ```
    
5. Récupérez vos fichiers dans le dossier `unpacked_fs`.
    

---

_Fin de la documentation_