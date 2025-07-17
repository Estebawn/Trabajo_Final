# Hotel StarGate
![image](https://github.com/user-attachments/assets/ff79ecc4-74f1-47f0-bf9a-efc92a83aea7)

## Integrantes
- Esteban Leon Duque - Encargado del análisis, investigación, documentación, codificación, tester.
  
## Vinculos academicos 
- Esteban Leon Duque - Estudiante de ingeniería industrial. A pesar de no tener ningún conocimiento o experiencia en programación, me esforzare por desarrollar mis habilidades en esta área y sacar el proyecto adelante con un buen estándar de calidad y eficiencia.

## Nombre del proyecto y detalles
- Descripción: : StarGate Hotel es un portal en la búsqueda de la armonía y confort brindando un servicio placentero y sereno basado en altos estándares para una estancia enriquecedora e inolvidable.
  
## Licencia del software 
https://creativecommons.org/publicdomain/zero/1.0/

## Reporte de vision
- En este proyecto se buscará desarrollar un sistema que permita un control eficiente y eficaz en el área administrativa por medio de la recolección de datos que faciliten la gestión de precios, cobros, facturas, reportes administrativos, reservas, ingresos y salidas.
  
## Especificacion de requisitos 
¿Qué puede hacer el sistema?

El sistema permitirá a los usuarios reservar habitaciones en el hotel, consultar disponibilidad en fechas específicas, y gestionar sus reservas

¿Qué se espera que se haga bien?

El sistema debe procesar las reservas de manera rápida y confiable evitando dobles reservas para la misma habitación en el mismo periodo de tiempo

¿Qué debe evitarse?

No debe permitir reservas para fechas pasadas ni aceptar datos inválidos

¿Qué funciones debe crear para resolver el problema planteado?

-	registrar usuarios
-	mostrar habitaciones disponibles
-	crear una reserva
- cancelar una reserva

## Plan de proyecto 

![image](https://github.com/user-attachments/assets/27395fc1-91c2-4e4c-a3a6-0854a046e0b8)

import os
import platform
import time
import re
import getpass
from datetime import datetime, timedelta
import matplotlib.pyplot as plt

# ============================
# Inicialización de Variables
# ============================
huespedes = []
reservas = []
habitaciones = {"Estandar": list(range(101, 111)), "Suite": list(range(201, 206))}  # 10 estándar, 5 suites
habitaciones_ocupadas = {}
admin_file = "admin.txt"
log_file = "log.txt"

# ============================
# Módulo de Registro de Log
# ============================
def inicializar_log():
    if not os.path.exists(log_file):
        with open(log_file, "w", encoding="utf-8") as f:
            try:
                usuario = getpass.getuser()
            except:
                usuario = "Desconocido"
            f.write(f"Usuario: {usuario}\n")
            f.write(f"Sistema Operativo: {platform.system()}\n")
            f.write(f"Plataforma: {platform.platform()}\n")
            f.write("Fecha y Hora | Acción | Tiempo de ejecución (s)\n")

def registrar_log(accion, inicio):
    duracion = round(time.time() - inicio, 6)
    tiempo_actual = datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f")[:-3]
    with open(log_file, "a", encoding="utf-8") as f:
        f.write(f"{tiempo_actual} | {accion} | {duracion}\n")

# ============================
# Registro de Huésped
# ============================
def registrar_huesped():
    inicio = time.time()
    nombre = input("Nombre: ")
    apellido = input("Apellido: ")
    doc = input("Documento de identidad: ")
    correo = input("Correo electrónico: ")
    telefono = input("Teléfono de contacto: ")

    if not (nombre.isalpha() and len(nombre) >= 3):
        print("Nombre inválido.")
        return
    if not (apellido.isalpha() and len(apellido) >= 3):
        print("Apellido inválido.")
        return
    if not (doc.isdigit() and 3 <= len(doc) <= 15):
        print("Documento inválido.")
        return
    if not re.match(r"[^@]+@[^@]+\.[^@]+", correo):
        print("Correo inválido.")
        return
    if not (telefono.isdigit() and 7 <= len(telefono) <= 15):
        print("Teléfono inválido.")
        return

    for h in huespedes:
        if h["doc"] == doc:
            print("Ya registrado.")
            return

    huespedes.append({"nombre": nombre, "apellido": apellido, "doc": doc, "correo": correo, "telefono": telefono})
    print("Huésped registrado con éxito.")
    registrar_log("Registrar huésped", inicio)

# ============================
# Realizar Reserva
# ============================
def reservar():
    inicio = time.time()
    doc = input("Documento del huésped: ")
    cliente = next((h for h in huespedes if h["doc"] == doc), None)
    if not cliente:
        print("Huésped no registrado.")
        return

    tipo = input("Tipo de habitación (Estandar/Suite): ").capitalize()
    if tipo not in habitaciones:
        print("Tipo inválido.")
        return

    fecha_ingreso = input("Fecha de ingreso (YYYY-MM-DD): ")
    try:
        ingreso = datetime.strptime(fecha_ingreso, "%Y-%m-%d")
    except:
        print("Fecha inválida.")
        return

    noches = input("Número de noches: ")
    if not noches.isdigit() or int(noches) <= 0:
        print("Número inválido.")
        return
    noches = int(noches)
    salida = ingreso + timedelta(days=noches)

    disponibles = [hab for hab in habitaciones[tipo] if hab not in habitaciones_ocupadas]
    if not disponibles:
        print("No hay habitaciones disponibles de ese tipo.")
        return

    num_hab = disponibles[0]
    habitaciones_ocupadas[num_hab] = (ingreso, salida)
    reservas.append({"doc": doc, "nombre": cliente["nombre"], "tipo": tipo, "hab": num_hab, "ingreso": ingreso, "salida": salida, "noches": noches})
    print("Reserva confirmada")
    print(f"Huésped: {cliente['nombre']}")
    print(f"Habitación: {tipo} #{num_hab}")
    print(f"Ingreso: {ingreso.date()} - Salida: {salida.date()}")
    print(f"Total estimado: ${noches * (120000 if tipo == 'Estandar' else 250000):,.0f}")
    registrar_log("Realizar reserva", inicio)

# ============================
# Registrar Salida (Check-out)
# ============================
def check_out():
    inicio = time.time()
    doc = input("Documento del huésped: ")
    reserva = next((r for r in reservas if r["doc"] == doc), None)
    if not reserva:
        print("No se encontró una reserva activa.")
        return

    hoy = datetime.now()
    noches = max(1, (hoy.date() - reserva["ingreso"].date()).days)
    precio_noche = 120000 if reserva["tipo"] == "Estandar" else 250000
    total = noches * precio_noche

    print("Factura de salida")
    print(f"Nombre: {reserva['nombre']}")
    print(f"Documento: {reserva['doc']}")
    print(f"Habitación: {reserva['tipo']} #{reserva['hab']}")
    print(f"Ingreso: {reserva['ingreso'].date()} - Salida: {hoy.date()}")
    print(f"Noches: {noches}")
    print(f"Total: ${total:,.0f}")

    habitaciones_ocupadas.pop(reserva["hab"])
    reservas.remove(reserva)
    registrar_log("Check-out", inicio)

# ============================
# Módulo de Administración
# ============================
def admin():
    inicio = time.time()
    if not os.path.exists(admin_file):
        with open(admin_file, "w") as f:
            f.write("admin,1234\n")

    import pandas as pd
    df = pd.read_csv(admin_file, names=["usuario", "clave"])

    usuario = input("Usuario admin: ")
    clave = input("Contraseña: ")

    if not ((df["usuario"] == usuario) & (df["clave"] == clave)).any():
        print("Acceso denegado.")
        return

    print("--- Reportes ---")
    print("Total huéspedes:", len(huespedes))
    print("Habitaciones ocupadas:", len(habitaciones_ocupadas))
    total_disponibles = sum(len(habs) for tipo, habs in habitaciones.items()) - len(habitaciones_ocupadas)
    print("Habitaciones disponibles:", total_disponibles)

    total_ingresos = 0
    noches_totales = 0
    historial = {}
    for r in reservas:
        precio = r["noches"] * (120000 if r["tipo"] == "Estandar" else 250000)
        total_ingresos += precio
        noches_totales += r["noches"]
        historial.setdefault(r["doc"], 0)
        historial[r["doc"]] += r["noches"]

    print("Ingresos totales: $", total_ingresos)
    if len(reservas) > 0:
        print("Promedio de estancia:", noches_totales / len(reservas))
    print("Hués. con + noches:", max(historial, key=historial.get, default="-"))
    print("Hués. con - noches:", min(historial, key=historial.get, default="-"))

    registrar_log("Acceso administración", inicio)

# ============================
# Menú Principal
# ============================
def menu():
    inicializar_log()
    while True:
        print("\n1. Registrar huésped")
        print("2. Realizar reserva")
        print("3. Check-out")
        print("4. Administración")
        print("5. Salir")
        op = input("Seleccione opción: ")

        if op == "1": registrar_huesped()
        elif op == "2": reservar()
        elif op == "3": check_out()
        elif op == "4": admin()
        elif op == "5": break
        else: print("Opción inválida")

menu()

