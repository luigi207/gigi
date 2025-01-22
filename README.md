import tkinter as tk
from tkinter import messagebox, PhotoImage
import json

# Percorso del file JSON
FILE_JSON = "users.json"

# Caricare i dati da JSON
def carica_dati():
    try:
        with open(FILE_JSON, "r") as file:
            return json.load(file)
    except FileNotFoundError:
        return {}  # Se il file non esiste, restituisce un dizionario vuoto
    except json.JSONDecodeError:
        return {}  # Se il file non è valido, restituisce un dizionario vuoto

# Salvare i dati su JSON
def salva_dati(dati):
    with open(FILE_JSON, "w") as file:
        json.dump(dati, file, indent=4)

class GymMember:
    def __init__(self, member_id, name, age, membership_type):
        self.member_id = member_id
        self.name = name
        self.age = age
        self.membership_type = membership_type

    def __str__(self):
        return f"ID: {self.member_id}, Nome: {self.name}, Età: {self.age}, Iscrizione: {self.membership_type}"

class GymManagement:
    def __init__(self):
        self.members = carica_dati()

    def add_member(self, member_id, name, age, membership_type):
        if member_id in self.members:
            return "L'ID del membro esiste già."
        else:
            self.members[member_id] = {
                "name": name,
                "age": age,
                "membership_type": membership_type
            }
            salva_dati(self.members)
            return "Membro aggiunto con successo."

    def remove_member(self, member_id):
        if member_id in self.members:
            del self.members[member_id]
            salva_dati(self.members)
            return "Membro rimosso con successo."
        else:
            return "ID del membro non trovato."

    def update_membership(self, member_id, new_membership_type):
        if new_membership_type not in ["giornaliera", "mensile", "annuale"]:
            return "Tipo di iscrizione non disponibile."

        if member_id in self.members:
            self.members[member_id]["membership_type"] = new_membership_type
            salva_dati(self.members)
            return "Iscrizione aggiornata con successo."
        else:
            return "ID del membro non trovato."

    def list_members(self):
        if not self.members:
            return "Nessun membro trovato."
        else:
            return "\n".join(
                f"ID: {member_id}, Nome: {data['name']}, Età: {data['age']}, Iscrizione: {data['membership_type']}"
                for member_id, data in self.members.items()
            )

class GymApp:
    def __init__(self, root):
        self.gym = GymManagement()
        self.root = root
        self.root.title("Stirati FK")
        self.root.configure(bg="black")
        self.root.resizable(False, False)

        self.logo = PhotoImage(file="image.png")
        self.logo = self.logo.subsample(4, 4)
        tk.Label(root, image=self.logo, bg="black").pack(pady=10)
        self.root.iconphoto(False, self.logo)

        tk.Label(root, text="--- Stirati FK ---", font=("Arial", 16), fg="yellow", bg="black").pack(pady=10)

        tk.Button(root, text="Aggiungi Membro", command=self.add_member_window, width=50, bg="yellow", fg="black").pack(pady=10)
        tk.Button(root, text="Rimuovi Membro", command=self.remove_member_window, width=50, bg="yellow", fg="black").pack(pady=10)
        tk.Button(root, text="Aggiorna Iscrizione", command=self.update_membership_window, width=50, bg="yellow", fg="black").pack(pady=10)
        tk.Button(root, text="Elenca Membri", command=self.list_members_window, width=50, bg="yellow", fg="black").pack(pady=10)
        tk.Button(root, text="Esci", command=root.quit, width=50, bg="yellow", fg="black").pack(pady=10)

    def add_member_window(self):
        window = tk.Toplevel(self.root)
        window.title("Aggiungi Membro")
        window.configure(bg="black")
        window.resizable(False, False)
        window.iconphoto(False, self.logo)

        tk.Label(window, text="ID Membro:", fg="yellow", bg="black").grid(row=0, column=0, padx=10, pady=5)
        member_id_entry = tk.Entry(window, bg="yellow", fg="black")
        member_id_entry.grid(row=0, column=1, padx=10, pady=5)

        tk.Label(window, text="Nome:", fg="yellow", bg="black").grid(row=1, column=0, padx=10, pady=5)
        name_entry = tk.Entry(window, bg="yellow", fg="black")
        name_entry.grid(row=1, column=1, padx=10, pady=5)

        tk.Label(window, text="Età:", fg="yellow", bg="black").grid(row=2, column=0, padx=10, pady=5)
        age_entry = tk.Entry(window, bg="yellow", fg="black")
        age_entry.grid(row=2, column=1, padx=10, pady=5)

        tk.Label(window, text="Tipo Iscrizione:", fg="yellow", bg="black").grid(row=3, column=0, padx=10, pady=5)
        membership_type_entry = tk.Entry(window, bg="yellow", fg="black")
        membership_type_entry.grid(row=3, column=1, padx=10, pady=5)

        def add_member_action():
            member_id = member_id_entry.get()
            name = name_entry.get()
            try:
                age = int(age_entry.get())
            except ValueError:
                messagebox.showerror("Errore", "Età non valida.")
                return
            membership_type = membership_type_entry.get()
            if membership_type not in ["giornaliera", "mensile", "annuale"]:
                messagebox.showerror("Errore", "Tipo di iscrizione non disponibile.")
                return
            result = self.gym.add_member(member_id, name, age, membership_type)
            messagebox.showinfo("Risultato", result)
            window.destroy()

        tk.Button(window, text="Aggiungi", command=add_member_action, bg="yellow", fg="black").grid(row=4, column=0, columnspan=2, pady=10)

    def remove_member_window(self):
        window = tk.Toplevel(self.root)
        window.title("Rimuovi Membro")
        window.configure(bg="black")
        window.resizable(False, False)
        window.iconphoto(False, self.logo)

        tk.Label(window, text="ID Membro:", fg="yellow", bg="black").grid(row=0, column=0, padx=10, pady=5)
        member_id_entry = tk.Entry(window, bg="yellow", fg="black")
        member_id_entry.grid(row=0, column=1, padx=10, pady=5)

        def remove_member_action():
            member_id = member_id_entry.get()
            result = self.gym.remove_member(member_id)
            messagebox.showinfo("Risultato", result)
            window.destroy()

        tk.Button(window, text="Rimuovi", command=remove_member_action, bg="yellow", fg="black").grid(row=1, column=0, columnspan=2, pady=10)

    def update_membership_window(self):
        window = tk.Toplevel(self.root)
        window.title("Aggiorna Iscrizione")
        window.configure(bg="black")
        window.resizable(False, False)
        window.iconphoto(False, self.logo)

        tk.Label(window, text="ID Membro:", fg="yellow", bg="black").grid(row=0, column=0, padx=10, pady=5)
        member_id_entry = tk.Entry(window, bg="yellow", fg="black")
        member_id_entry.grid(row=0, column=1, padx=10, pady=5)

        tk.Label(window, text="Nuovo Tipo Iscrizione:", fg="yellow", bg="black").grid(row=1, column=0, padx=10, pady=5)
        membership_type_entry = tk.Entry(window, bg="yellow", fg="black")
        membership_type_entry.grid(row=1, column=1, padx=10, pady=5)

        def update_membership_action():
            member_id = member_id_entry.get()
            new_membership_type = membership_type_entry.get()
            result = self.gym.update_membership(member_id, new_membership_type)
            messagebox.showinfo("Risultato", result)
            window.destroy()

        tk.Button(window, text="Aggiorna", command=update_membership_action, bg="yellow", fg="black").grid(row=2, column=0, columnspan=2, pady=10)

    def list_members_window(self):
        window = tk.Toplevel(self.root)
        window.title("Elenco Membri")
        window.configure(bg="black")
        window.resizable(False, False)
        window.iconphoto(False, self.logo)

        members = self.gym.list_members()
        tk.Label(window, text=members, justify="left", anchor="w", fg="yellow", bg="black").pack(padx=10, pady=10)

class AuthenticationApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Login")
        self.root.configure(bg="black")
        self.root.resizable(False, False)

        self.logo = PhotoImage(file="image.png")
        self.logo = self.logo.subsample(4, 4)
        tk.Label(root, image=self.logo, bg="black").pack(pady=10)

        tk.Label(root, text="Inserisci PIN", font=("Arial", 16), fg="yellow", bg="black").pack(pady=10)

        self.pin_entry = tk.Entry(root, show="*", justify="center", font=("Arial", 16), bg="yellow", fg="black")
        self.pin_entry.pack(pady=10)

        tk.Button(root, text="Login", command=self.check_pin, bg="yellow", fg="black").pack(pady=10)

    def check_pin(self):
        if self.pin_entry.get() == "1234":
            self.root.destroy()
            main_root = tk.Tk()
            GymApp(main_root)
            main_root.mainloop()
        else:
            messagebox.showerror("Errore", "PIN non valido!")

if __name__ == "__main__":
    root = tk.Tk()
    AuthenticationApp(root)
    root.mainloop()
