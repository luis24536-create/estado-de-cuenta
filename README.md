# estado-de-cuenta
import tkinter as tk
from tkinter import simpledialog, messagebox
from datetime import date

ventana = tk.Tk()
ventana.title("Estado de cuenta")
ventana.geometry("650x550")

class Cliente:
    def __init__(self, nombre, apellido1, apellido2, fecha_nac, domicilio):
        self.nombre = nombre
        self.apellido1 = apellido1
        self.apellido2 = apellido2
        self.fecha_nac = fecha_nac
        self.domicilio = domicilio

class Cuenta:
    def __init__(self, numero, saldo_inicial):
        self.numero = numero
        self.saldo = saldo_inicial

    def abonar(self, monto):
        self.saldo += monto

    def cargar(self, monto):
        if self.saldo >= monto:
            self.saldo -= monto
            return True
        else:
            return False

class Movimiento:
    def __init__(self, fecha_mov, descripcion, tipo, cargo, abono, saldo):
        self.fecha_mov = fecha_mov
        self.descripcion = descripcion
        self.tipo = tipo
        self.cargo = cargo
        self.abono = abono
        self.saldo = saldo

class EstadoCuenta:
    def __init__(self, cliente, cuenta, fecha_ingreso, movimientos=None):
        if movimientos is None:
            movimientos = []
        self.cliente = cliente
        self.cuenta = cuenta
        self.fecha_ingreso = fecha_ingreso
        self.movimientos = movimientos

    def agregar_movimiento(self, fecha, descripcion, tipo, monto):
        abono = 0
        cargo = 0
        tipos_abono = ["Abono", "Transferencia recibida", "Depósito"]
        tipos_cargo = ["Cargo", "Pago servicio", "Transferencia enviada", "Retiro"]

        if tipo in tipos_abono:
            abono = monto
        elif tipo in tipos_cargo:
            cargo = monto
        else:
            abono = monto  

        if cargo > 0 and cargo > self.cuenta.saldo:
            return False

        self.cuenta.saldo += abono
        self.cuenta.saldo -= cargo

        mov = Movimiento(fecha, descripcion, tipo, cargo, abono, self.cuenta.saldo)
        self.movimientos.append(mov)
        return True

def mostrar_estado_cuenta():
    for widget in ventana.winfo_children():
        widget.destroy()

    tk.Label(ventana, text=f"Cliente: {estado_cuenta.cliente.nombre} {estado_cuenta.cliente.apellido1} {estado_cuenta.cliente.apellido2}", font=("Arial", 14)).pack(pady=10)
    tk.Label(ventana, text=f"Domicilio: {estado_cuenta.cliente.domicilio}", font=("Arial", 12)).pack(pady=5)
    tk.Label(ventana, text=f"Fecha de nacimiento: {estado_cuenta.cliente.fecha_nac}", font=("Arial", 12)).pack(pady=5)
    tk.Label(ventana, text=f"Número de cuenta: {estado_cuenta.cuenta.numero}", font=("Arial", 12)).pack(pady=5)
    tk.Label(ventana, text=f"Saldo actual: {estado_cuenta.cuenta.saldo:.2f}", font=("Arial", 12, "bold")).pack(pady=10)

    tk.Label(ventana, text="Movimientos:", font=("Arial", 13, "underline")).pack(pady=5)

    for mov in estado_cuenta.movimientos:
        texto = f"Fecha: {mov.fecha_mov} | Tipo: {mov.tipo} | Desc: {mov.descripcion} | Cargo: {mov.cargo:.2f} | Abono: {mov.abono:.2f} | Saldo: {mov.saldo:.2f}"
        tk.Label(ventana, text=texto, font=("Arial", 11)).pack(anchor="w", padx=20)

    tk.Button(ventana, text="Agregar Movimiento", command=abrir_dialogo_movimiento, font=("Arial", 12), bg="lightgreen").pack(pady=20)

def abrir_dialogo_movimiento():
    tipos_movimiento = ["Abono", "Cargo", "Transferencia recibida", "Transferencia enviada", "Pago servicio", "Depósito", "Retiro"]

    tipo = simpledialog.askstring("Tipo de movimiento", f"Selecciona el tipo de movimiento:\n{', '.join(tipos_movimiento)}")
    if tipo is None:
        return  

    tipo = tipo.strip()
    if tipo not in tipos_movimiento:
        messagebox.showerror("Error", "Tipo de movimiento no válido.")
        return

    descripcion = simpledialog.askstring("Descripción", "Ingresa la descripción del movimiento:")
    if descripcion is None or descripcion.strip() == "":
        messagebox.showerror("Error", "Descripción obligatoria.")
        return

    monto_str = simpledialog.askstring("Monto", "Ingresa el monto:")
    if monto_str is None:
        return

    try:
        monto = float(monto_str)
        if monto <= 0:
            raise ValueError()
    except:
        messagebox.showerror("Error", "Monto inválido, debe ser un número mayor que 0.")
        return

    ok = estado_cuenta.agregar_movimiento(date.today(), descripcion, tipo, monto)
    if not ok:
        messagebox.showerror("Error", "Saldo insuficiente para realizar este cargo.")
        return

    mostrar_estado_cuenta()

def crear_estado():
    nombre = entry_nombre.get()
    apellido1 = entry_apellido1.get()
    apellido2 = entry_apellido2.get()
    fecha_nac = entry_fecha_nac.get()
    domicilio = entry_domicilio.get()
    numero_cuenta = entry_num_cuenta.get()
    saldo_inicial = entry_saldo.get()

    try:
        saldo_inicial = float(saldo_inicial)
        if saldo_inicial < 0:
            messagebox.showerror("Error", "Saldo inicial no puede ser negativo.")
            return
    except ValueError:
        messagebox.showerror("Error", "Saldo inicial debe ser un número.")
        return

    if not (nombre and apellido1 and apellido2 and fecha_nac and domicilio and numero_cuenta):
        messagebox.showerror("Error", "Por favor, complete todos los campos.")
        return

    global estado_cuenta
    cliente = Cliente(nombre, apellido1, apellido2, fecha_nac, domicilio)
    cuenta = Cuenta(numero_cuenta, saldo_inicial)
    estado_cuenta = EstadoCuenta(cliente, cuenta, date.today())

    mostrar_estado_cuenta()

tk.Label(ventana, text="Nombre:", font=("Arial", 12)).pack()
entry_nombre = tk.Entry(ventana, font=("Arial", 12))
entry_nombre.pack()

tk.Label(ventana, text="Apellido Paterno:", font=("Arial", 12)).pack()
entry_apellido1 = tk.Entry(ventana, font=("Arial", 12))
entry_apellido1.pack()

tk.Label(ventana, text="Apellido Materno:", font=("Arial", 12)).pack()
entry_apellido2 = tk.Entry(ventana, font=("Arial", 12))
entry_apellido2.pack()

tk.Label(ventana, text="Fecha de nacimiento:", font=("Arial", 12)).pack()
entry_fecha_nac = tk.Entry(ventana, font=("Arial", 12))
entry_fecha_nac.pack()

tk.Label(ventana, text="Domicilio:", font=("Arial", 12)).pack()
entry_domicilio = tk.Entry(ventana, font=("Arial", 12))
entry_domicilio.pack()

tk.Label(ventana, text="Número de cuenta:", font=("Arial", 12)).pack()
entry_num_cuenta = tk.Entry(ventana, font=("Arial", 12))
entry_num_cuenta.pack()

tk.Label(ventana, text="Saldo inicial:", font=("Arial", 12)).pack()
entry_saldo = tk.Entry(ventana, font=("Arial", 12))
entry_saldo.pack()

tk.Button(ventana, text="Crear Estado de Cuenta", command=crear_estado, font=("Arial", 12), bg="lightblue").pack(pady=15)

ventana.mainloop()
