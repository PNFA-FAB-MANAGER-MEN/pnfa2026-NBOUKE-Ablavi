# Journal du mercredi le 16 Septembre 2026 rédigé par : Ablavi N'BOUKE

## Fait aujourd'hui
En cette journée du 16 septembre 2026, je me suis concentré sur la partie soudure de notre projet:coffre-fort inteligent
En cette journée, notre équipe s'est concentrée sur le système d'alimentation autonome, la résolution des pannes du circuit et la finition esthétique du prototype de **Coffre-fort intelligent**.

* **Recherche de matériel** :
* Recherche et approvisionnement des ùateriels necessaire pour la soudure: fer à souder; etain, fils de connexion, graise 
 [soudure finale] (photo : ../medias/2026-09-16 at 21.56.27.jpeg)
- validation du code pour l'ensemble du systeme
le code finale de notre cooffre-fort Intelligent est le suivant
## Brochage Matériel

| Composant | Pin ESP32 | Type |
| --- | --- | --- |
| LCD SDA | GPIO 21 | I2C |
| LCD SCL | GPIO 22 | I2C |
| Relais / Solénoïde | GPIO 26 | OUT |
| Clavier Rows | 13, 12, 14, 27 | OUT |
| Clavier Cols | 33, 32, 15, 4 | IN PULL_DOWN |

**Règle de sécurité:** `relais.value(0)` = Verrouillé au démarrage.

### 3. Code Source Principal (main.py)

Ce code fonctionne et a été testé sur ESP32.

```python
from machine import Pin, I2C
import time
from esp32_i2c_lcd import I2cLcd

# =========================================================
# CONFIGURATION
# =========================================================
CODE_PIN_SECRET = "1234"
DUREE_DEVERROUILLAGE = 3  # en secondes

# 1. INITIALISATION MATÉRIELLE
i2c = I2C(0, scl=Pin(22), sda=Pin(21), freq=400000)
lcd = I2cLcd(i2c, 0x27, 2, 16)

# Relais / Solénoïde sur GPIO 26
relais = Pin(26, Pin.OUT)

# REGLE STRICTE : 0 = VERROUILLÉ AU DÉMARRAGE
relais.value(0)

# Clavier Matriciel 4x4
row_pins = [Pin(13, Pin.OUT), Pin(12, Pin.OUT), Pin(14, Pin.OUT), Pin(27, Pin.OUT)]
col_pins = [Pin(33, Pin.IN, Pin.PULL_DOWN), Pin(32, Pin.IN, Pin.PULL_DOWN), 
            Pin(15, Pin.IN, Pin.PULL_DOWN), Pin(4, Pin.IN, Pin.PULL_DOWN)]

keys = [
    ['1', '2', '3', 'A'],
    ['4', '5', '6', 'B'],
    ['7', '8', '9', 'C'],
    ['*', '0', '#', 'D']
]

def scan_keypad():
    for r_idx, r_pin in enumerate(row_pins):
        r_pin.value(1)
        for c_idx, c_pin in enumerate(col_pins):
            if c_pin.value() == 1:
                r_pin.value(0)
                return keys[r_idx][c_idx]
        r_pin.value(0)
    return None

def reinitialiser_ecran():
    lcd.clear()
    lcd.move_to(0, 0)
    lcd.putstr("Saisir le Code:")
    lcd.move_to(0, 1)
    lcd.putstr("> ")

# Initialisation
reinitialiser_ecran()
code_saisi = ""
derniere_touche = ""

while True:
    touche = scan_keypad()
    
    if touche is not None:
        if touche != derniere_touche:
            
            # --- VALIDER AVEC '#' ---
            if touche == '#':
                if code_saisi == CODE_PIN_SECRET:
                    lcd.clear()
                    lcd.move_to(0, 0)
                    lcd.putstr("Code Correct !")
                    lcd.move_to(0, 1)
                    lcd.putstr("Porte Ouverte")
                    
                    # 1 = DÉVERROUILLER
                    relais.value(1)
                    print("Relais = 1 (Déverrouillé)")
                    
                    time.sleep(DUREE_DEVERROUILLAGE)
                    
                    # 0 = VERROUILLER
                    relais.value(0)
                    print("Relais = 0 (Verrouillé)")
                else:
                    lcd.clear()
                    lcd.move_to(0, 0)
                    lcd.putstr("Code Faux !")
                    lcd.move_to(0, 1)
                    lcd.putstr("Acces Refuse")
                    time.sleep(2)
                
                code_saisi = ""
                reinitialiser_ecran()

            # --- EFFACER AVEC '*' ---
            elif touche == '*':
                code_saisi = ""
                reinitialiser_ecran()

            # --- SAISIE DES CHIFFRES ---
            else:
                if len(code_saisi) < 10:
                    code_saisi += touche
                    lcd.move_to(0, 1)
                    lcd.putstr("> " + "*" * len(code_saisi) + "      ")

            derniere_touche = touche
            time.sleep(0.2)
    else:
        derniere_touche = ""
        
    time.sleep(0.05)

