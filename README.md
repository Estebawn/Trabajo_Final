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

## Codigo
import re
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
import pandas as pd

huespedes = {}
habitaciones = {
    "Estándar": [{"numero": 101, "reservas": []}, {"numero": 102, "reservas": []}],
    "Suite": [{"numero": 201, "reservas": []}]
}
historial = []
checkouts = []
precios = {"Estándar": 120000, "Suite": 250000}

def validar_nombre(nombre):
    return len(nombre) >= 3 and nombre.isalpha()

def validar_documento(doc):
    return doc.isdigit() and 3 <= len(doc) <= 15

def validar_correo(correo):
    return re.match(r"^[\w\.-]+@[\w\.-]+\.\w+$", correo)

def validar_telefono(tel):
    return tel.isdigit() and 7 <= len(tel) <= 15

def registrar_huesped():
    print("Registro de huésped")
    nombre = input("Nombre: ").strip()
    apellido = input("Apellido: ").strip()
    doc = input("Documento: ").strip()
    correo = input("Correo: ").strip()
    tel = input("Teléfono: ").strip()
    errores = []
    if not validar_nombre(nombre): errores.append("Nombre inválido")
    if not validar_nombre(apellido): errores.append("Apellido inválido")
    if not validar_documento(doc): errores.append("Documento inválido")
    if not validar_correo(correo): errores.append("Correo inválido")
    if not validar_telefono(tel): errores.append("Teléfono inválido")
    if errores:
        print("Errores:")
        for e in errores: print("-", e)
    else:
        huespedes[doc] = {"nombre": nombre, "apellido": apellido, "correo": correo, "tel": tel}
        print("Huésped registrado")

def esta_disponible(hab, ingreso, noches):
    salida = ingreso + timedelta(days=noches)
    for r in hab["reservas"]:
        if not (salida <= r["ingreso"] or ingreso >= r["salida"]):
            return False
    return True

def realizar_reserva():
    print("Reserva de habitación")
    doc = input("Documento del huésped: ").strip()
    if doc not in huespedes:
        print("Huésped no registrado")
        return
    tipo = input("Tipo de habitación (Estándar/Suite): ").capitalize()
    if tipo not in habitaciones:
        print("Tipo inválido")
        return
    try:
        fecha_str = input("Fecha de ingreso (YYYY-MM-DD): ")
        ingreso = datetime.strptime(fecha_str, "%Y-%m-%d").date()
        noches = int(input("Noches: "))
    except:
        print("Fecha o número de noches inválidos")
        return
    for hab in habitaciones[tipo]:
        if esta_disponible(hab, ingreso, noches):
            salida = ingreso + timedelta(days=noches)
            hab["reservas"].append({"doc": doc, "ingreso": ingreso, "salida": salida, "noches": noches})
            print("Reserva confirmada")
            print("Huésped:", huespedes[doc]["nombre"], huespedes[doc]["apellido"])
            print("Hab:", tipo, hab["numero"])
            print("Ingreso:", ingreso, "Salida:", salida)
            print("Noches:", noches)
            print("Total: $", noches * precios[tipo])
            return
    print("No hay disponibilidad")

def check_out():
    print("Check-Out")
    doc = input("Documento: ").strip()
    for tipo in habitaciones:
        for hab in habitaciones[tipo]:
            for r in hab["reservas"]:
                if r["doc"] == doc:
                    ingreso = r["ingreso"]
                    hoy = datetime.now().date()
                    noches = (hoy - ingreso).days
                    if noches < 1: noches = 1
                    total = noches * precios[tipo]
                    print("Factura")
                    print("Huésped:", huespedes[doc]["nombre"], huespedes[doc]["apellido"])
                    print("Doc:", doc)
                    print("Tipo:", tipo, "Hab:", hab["numero"])
                    print("Ingreso:", ingreso, "Salida:", hoy)
                    print("Noches:", noches)
                    print("Total: $", total)
                    checkouts.append({"doc": doc, "nombre": huespedes[doc]["nombre"], "apellido": huespedes[doc]["apellido"],
                                      "tipo": tipo, "hab": hab["numero"], "ingreso": ingreso, "salida": hoy,
                                      "noches": noches, "total": total})
                    historial.append(checkouts[-1])
                    hab["reservas"].remove(r)
                    return
    print("No se encontró una reserva activa")

def login_admin():
    try:
        df = pd.read_csv("admins.txt", header=None, names=["usuario", "clave"])
    except:
        print("Archivo admins.txt no encontrado")
        return False
    u = input("Usuario: ").strip()
    c = input("Contraseña: ").strip()
    return ((df["usuario"] == u) & (df["clave"] == c)).any()

def reportes():
    df = pd.DataFrame(historial)
    if df.empty:
        print("No hay datos")
        return
    print("Total huéspedes:", len(huespedes))
    ocupadas = sum(len(h["reservas"]) for h in habitaciones["Estándar"] + habitaciones["Suite"])
    disponibles = sum(len(h) for h in habitaciones.values()) - ocupadas
    print("Habitaciones ocupadas:", ocupadas)
    print("Habitaciones disponibles:", disponibles)
    print("Ingresos totales: $", df["total"].sum())
    print("Tiempo promedio de estancia:", df["noches"].mean())
    print("Huésped con más noches:", df.groupby("doc")["noches"].sum().idxmax())
    print("Huésped con menos noches:", df.groupby("doc")["noches"].sum().idxmin())
    print("\nHistorial de huéspedes:")
    print(df[["doc", "nombre", "apellido", "noches", "total"]])

    # Gráficos
    df.groupby("tipo").size().plot(kind="bar", title="Habitaciones ocupadas por tipo")
    plt.show()
    plt.pie([ocupadas, disponibles], labels=["Ocupadas", "Disponibles"], autopct="%1.1f%%")
    plt.title("Ocupación actual")
    plt.show()
    df.groupby("salida").size().plot(kind="line", title="Check-Outs por día")
    plt.show()
    df.groupby("doc")["noches"].sum().nlargest(10).plot(kind="barh", title="Top 10 noches hospedadas")
    plt.show()
    plt.scatter(df["noches"], df["total"])
    plt.title("Noches vs Total Pagado")
    plt.xlabel("Noches")
    plt.ylabel("Total")
    plt.show()
    df.groupby("tipo")["total"].sum().plot(kind="pie", autopct="%1.1f%%", title="Ingresos por tipo")
    plt.ylabel("")
    plt.show()
    df["noches"].plot(kind="hist", bins=5, title="Distribución de noches")
    plt.show()
    df.groupby("salida")[["total"]].sum().plot(kind="bar", title="Ingresos diarios")
    plt.twinx()
    df.groupby("salida").size().plot(color="red", marker="o", label="Huéspedes")
    plt.title("Ingresos y huéspedes por día")
    plt.legend()
    plt.show()

def menu():
    while True:
        print("\nMenú:")
        print("1. Registrar huésped")
        print("2. Reservar habitación")
        print("3. Registrar salida")
        print("4. Acceso administrador")
        print("5. Salir")
        op = input("Opción: ")
        if op == "1":
            registrar_huesped()
        elif op == "2":
            realizar_reserva()
        elif op == "3":
            check_out()
        elif op == "4":
            if login_admin():
                reportes()
            else:
                print("Acceso denegado")
        elif op == "5":
            break
        else:
            print("Opción inválida")

menu()

