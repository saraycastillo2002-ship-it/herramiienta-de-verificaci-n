import tkinter as tk
from tkinter import messagebox
import csv
import os
import random
from datetime import datetime

# --- Sensor simulado (reemplaza a sense_emu) ---
class SenseHat:
    def get_temperature(self):
        return random.uniform(20, 30)

    def get_humidity(self):
        return random.uniform(40, 70)

    def get_pressure(self):
        return random.uniform(1000, 1020)

sense = SenseHat()
root = tk.Tk()
root.title("Panel IoT - Captura Automática")
root.geometry("350x380")

CARPETA = r"C:\practica"
ARCHIVO = os.path.join(CARPETA, "registro_tarea2_automatico.csv")
INTERVALO_MS = 5000  # 5 segundos

temp_actual = 0.0
hum_actual = 0.0
pres_actual = 0.0
automatico = True  # indica si la captura automática está activa


def actualizar_datos():
    global temp_actual, hum_actual, pres_actual
    temp_actual = sense.get_temperature()
    hum_actual = sense.get_humidity()
    pres_actual = sense.get_pressure()
    lbl_temp.config(text=f"Temperatura: {temp_actual:.1f} °C")
    lbl_hum.config(text=f"Humedad: {hum_actual:.1f} %")
    lbl_pres.config(text=f"Presión: {pres_actual:.1f} hPa")
    lbl_estado.config(text="Datos actualizados", fg="blue")


def guardar_datos(silencioso=False):
    tiempo_actual = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    try:
        os.makedirs(CARPETA, exist_ok=True)
        escribir_encabezado = (not os.path.exists(ARCHIVO)) or os.path.getsize(ARCHIVO) == 0
        with open(ARCHIVO, mode='a', newline='', encoding='utf-8') as archivo:
            writer = csv.writer(archivo)
            if escribir_encabezado:
                writer.writerow(["Fecha/Hora", "Temperatura (°C)", "Humedad (%)", "Presión (hPa)"])
            writer.writerow([tiempo_actual, round(temp_actual, 2),
                             round(hum_actual, 2), round(pres_actual, 2)])
        lbl_estado.config(text=f"Guardado a las {tiempo_actual[-8:]}", fg="green")
        if not silencioso:
            messagebox.showinfo("Éxito", "¡Datos guardados correctamente en CSV/Excel!")
    except Exception as e:
        messagebox.showerror("Error", f"No se pudo guardar el archivo: {e}")


def captura_automatica():
    # Se ejecuta cada 5 segundos mientras el modo automático esté activo
    if automatico:
        actualizar_datos()
        guardar_datos(silencioso=True)
        root.after(INTERVALO_MS, captura_automatica)


def alternar_automatico():
    global automatico
    if automatico:
        automatico = False
        btn_auto.config(text="Reanudar captura automática", bg="#4CAF50")
        lbl_estado.config(text="Captura automática detenida", fg="red")
    else:
        automatico = True
        btn_auto.config(text="Detener captura automática", bg="#F44336")
        captura_automatica()


# --- Interfaz ---
lbl_titulo = tk.Label(root, text="Estación IoT - Captura cada 5 s", font=("Arial", 14, "bold"))
lbl_titulo.pack(pady=10)

lbl_temp = tk.Label(root, text="Temperatura: -- °C", font=("Arial", 12))
lbl_temp.pack(pady=5)

lbl_hum = tk.Label(root, text="Humedad: -- %", font=("Arial", 12))
lbl_hum.pack(pady=5)

lbl_pres = tk.Label(root, text="Presión: -- hPa", font=("Arial", 12))
lbl_pres.pack(pady=5)

btn_auto = tk.Button(root, text="Detener captura automática", command=alternar_automatico,
                     bg="#F44336", fg="white", width=28)
btn_auto.pack(pady=10)

lbl_estado = tk.Label(root, text="Iniciando...", font=("Arial", 10, "italic"), fg="gray")
lbl_estado.pack(pady=10)

captura_automatica()
root.mainloop()
