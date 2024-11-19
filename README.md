import flet as ft

def main(page: ft.Page):
    # Variables locales para el estado de la aplicación
    gasto_basico = None
    deudas = []  # Lista para almacenar las deudas
    total = None
    sueldo = None
    cantidad_deudas = 0
    total_pagos_mensuales = 0  # Suma total de los pagos mensuales

    # Título de la página
    page.title = "Calculadora de Sueldo y Gasto"
    page.vertical_alignment = ft.MainAxisAlignment.CENTER
    page.horizontal_alignment = ft.CrossAxisAlignment.CENTER  # Centra todo en el eje horizontal
    page.padding = ft.Padding(top=20, right=20, bottom=20, left=20)  # Agrega relleno (padding) en todos los lados

    # Función para manejar la acción del botón "Volver al formulario"
    def volver_formulario(e):
        deudas_seccion.visible = False
        formulario_seccion.visible = True
        plan_seccion.visible = False
        alerta_seccion.visible = False
        page.update()

    # Función para manejar la acción del botón "Guardar Gasto"
    def guardar_gasto(e):
        nonlocal gasto_basico, total, sueldo  # Usamos nonlocal para modificar variables de la función main
        try:
            sueldo = float(sueldo_input.value)
            gasto = gasto_input.value.strip()

            if '.' in gasto:
                mensaje.value = "Por favor, introduce un número entero (sin decimales)."
                page.update()
                return

            if gasto_basico is None:
                gasto_basico = int(gasto)
                total = sueldo - gasto_basico
                mensaje.value = f"Gasto en necesidades básicas registrado: {gasto_basico}. Tu sueldo es {round(sueldo)}."
                total_text.value = f"Total restante: {round(total)}"
                total_text.size = 24
                total_text.weight = ft.FontWeight.BOLD
            else:
                mensaje.value = f"Ya se ha registrado un gasto en necesidades básicas: {gasto_basico}."
                total_text.value = ""

            gasto_input.value = ""
            page.update()

        except ValueError:
            mensaje.value = "Por favor, introduce un valor numérico válido."
            page.update()

    # Función para mostrar el plan recomendado
    def ver_plan_recomendado(e):
        plan_seccion.visible = True
        formulario_seccion.visible = False
        deudas_seccion.visible = False
        alerta_seccion.visible = False

        if sueldo is not None and gasto_basico is not None:
            necesidades_basicas = round(sueldo * 0.50)
            deseos = round(sueldo * 0.30)
            ahorros = round(sueldo * 0.20)

            plan_necesidades.value = f"50% para necesidades básicas: {necesidades_basicas}"
            plan_deseos.value = f"30% para deseos o gastos hormiga: {deseos}"
            plan_ahorros.value = f"20% para ahorrar: {ahorros}"

        page.update()

    # Función para manejar la visibilidad del campo de cantidad de deudas
    def on_deudas_change(e):
        cantidad_deudas_input.visible = tiene_deudas_checkbox.value
        if not tiene_deudas_checkbox.value:
            deuda_inputs.clear()
            meses_inputs.clear()
        page.update()

    # Función para ir a la sección de deudas
    def agregar_deudas(e):
        nonlocal cantidad_deudas
        try:
            cantidad_deudas = int(cantidad_deudas_input.value)
            deuda_inputs.clear()
            meses_inputs.clear()

            for i in range(cantidad_deudas):
                deuda_inputs.append(ft.TextField(label=f"Valor de la deuda {i + 1}", keyboard_type=ft.KeyboardType.NUMBER))
                meses_inputs.append(ft.TextField(label=f"Meses a pagar de la deuda {i + 1}", keyboard_type=ft.KeyboardType.NUMBER))

            deudas_seccion.controls.extend(deuda_inputs + meses_inputs)
            deudas_seccion.visible = True
            formulario_seccion.visible = False
            page.update()

        except ValueError:
            mensaje.value = "Por favor, introduce un número válido para la cantidad de deudas."
            page.update()

    # Función para guardar las deudas ingresadas
    def guardar_deudas(e):
        nonlocal deudas, total_pagos_mensuales, total
        deudas.clear()
        total_pagos_mensuales = 0  # Reiniciar la suma de pagos mensuales

        try:
            for i in range(cantidad_deudas):
                deuda = float(deuda_inputs[i].value)
                meses = int(meses_inputs[i].value)

                if deuda <= 0 or meses <= 0:
                    mensaje.value = "Por favor, ingresa valores positivos para la deuda y los meses."
                    page.update()
                    return

                # Calcular el pago mensual para la deuda
                pago_mensual = round(deuda / meses, 2)
                total_pagos_mensuales += pago_mensual

                # Almacenar la deuda y el pago mensual
                deudas.append({"deuda": deuda, "meses": meses, "pago_mensual": pago_mensual})

            mensaje.value = "Deudas registradas con éxito."
            deudas_text.value = f"Deudas registradas: {len(deudas)}"
            deudas_seccion.visible = False
            formulario_seccion.visible = True

            # Actualizar la visualización de las deudas debajo del total restante
            deuda_list_text.controls.clear()
            for i, deuda in enumerate(deudas, start=1):
                deuda_list_text.controls.append(
                    ft.Text(f"Deuda {i}: ${deuda['deuda']} a pagar en {deuda['meses']} meses - Pago mensual: ${deuda['pago_mensual']}")
                )
            
            # Deshabilitar la opción de agregar más deudas
            tiene_deudas_checkbox.disabled = True
            cantidad_deudas_input.disabled = True
            agregar_deudas_button.disabled = True

            # Verificar si la suma de pagos mensuales excede el 30% del total restante
            maximo_permitido = total * 0.30
            if total_pagos_mensuales > maximo_permitido:
                alerta_button.visible = True
            else:
                alerta_button.visible = False

            # Restar los pagos mensuales del total restante
            total -= total_pagos_mensuales
            total_text.value = f"Total restante después de pagos: {round(total)}"
            page.update()

        except ValueError:
            mensaje.value = "Por favor, ingresa un número válido para la deuda y los meses."
            page.update()

    # Función para manejar la acción del botón de alerta
    def mostrar_alerta(e):
        alerta_seccion.visible = True
        formulario_seccion.visible = False
        deudas_seccion.visible = False
        plan_seccion.visible = False
        alerta_text.value = f"¡ALERTA! Tus pagos mensuales (${total_pagos_mensuales}) exceden el 30% del total restante (${round(total * 0.30)})."
        page.update()

    # Función para reiniciar la aplicación
    def reiniciar_aplicacion(e):
        nonlocal gasto_basico, deudas, total, sueldo, cantidad_deudas, total_pagos_mensuales
        # Reiniciar todas las variables a su estado inicial
        gasto_basico = None
        deudas.clear()
        total = None
        sueldo = None
        cantidad_deudas = 0
        total_pagos_mensuales = 0

        # Limpiar todos los campos y elementos de la interfaz
        sueldo_input.value = ""
        gasto_input.value = ""
        mensaje.value = ""
        total_text.value = ""
        deuda_list_text.controls.clear()
        tiene_deudas_checkbox.value = False
        cantidad_deudas_input.value = ""
        cantidad_deudas_input.visible = False
        
        # Limpiar las entradas de las deudas anteriores
        deuda_inputs.clear()
        meses_inputs.clear()
        deudas_seccion.controls = [deudas_text, ft.ElevatedButton("Guardar deudas", on_click=guardar_deudas)]
        deudas_seccion.visible = False
        alerta_button.visible = False

        # Volver al formulario inicial
        formulario_seccion.visible = True
        plan_seccion.visible = False
        alerta_seccion.visible = False

        page.update()

    # Definir los componentes de la interfaz de usuario
    sueldo_input = ft.TextField(label="Introduce tu sueldo", keyboard_type=ft.KeyboardType.NUMBER)
    gasto_input = ft.TextField(label="Introduce tu gasto básico", keyboard_type=ft.KeyboardType.NUMBER)
    mensaje = ft.Text()
    total_text = ft.Text()

    # Contenedor para mostrar las deudas registradas
    deuda_list_text = ft.Column()

    # Logo y nombre de la aplicación
    header = ft.Row(
        [ft.Icon(ft.icons.MONEY), ft.Text("MoneyPlan", size=24, weight=ft.FontWeight.BOLD)],
        alignment=ft.MainAxisAlignment.START,
        spacing=5
    )

    plan_necesidades = ft.Text("50% para necesidades básicas: 0", size=18)
    plan_deseos = ft.Text("30% para deseos o gastos hormiga: 0", size=18)
    plan_ahorros = ft.Text("20% para ahorrar o pagar deudas: 0", size=18)

    plan_seccion = ft.Column(
        [
            ft.Text("Plan recomendado", size=30, weight=ft.FontWeight.BOLD),
            plan_necesidades,
            plan_deseos,
            plan_ahorros,
            ft.ElevatedButton("Volver al formulario", on_click=volver_formulario)
        ],
        visible=False
    )

    tiene_deudas_checkbox = ft.Checkbox("¿Tienes deudas?", on_change=on_deudas_change)
    cantidad_deudas_input = ft.TextField(label="¿Cuántas deudas tienes?", keyboard_type=ft.KeyboardType.NUMBER, visible=False)
    agregar_deudas_button = ft.ElevatedButton("Agregar deudas", on_click=agregar_deudas)
    deuda_inputs = []
    meses_inputs = []

    deudas_text = ft.Text()
    deudas_seccion = ft.Column(
        [
            ft.Text("Deudas", size=30, weight=ft.FontWeight.BOLD),
            deudas_text,
            ft.ElevatedButton("Guardar deudas", on_click=guardar_deudas),
            ft.ElevatedButton("Volver al formulario", on_click=volver_formulario)
        ],
        visible=False
    )

    alerta_button = ft.ElevatedButton(
        "¡ALERTA! Tus pagos mensuales son demasiado altos",
        on_click=mostrar_alerta,
        visible=False,
        color=ft.colors.RED
    )

    alerta_text = ft.Text("", size=24, weight=ft.FontWeight.BOLD, color=ft.colors.RED)
    alerta_seccion = ft.Column(
        [
            ft.Text("ALERTA", size=30, weight=ft.FontWeight.BOLD, color=ft.colors.RED),
            alerta_text,
            ft.ElevatedButton("Volver al formulario", on_click=volver_formulario)
        ],
        visible=False
    )

    # Botón de reinicio
    reinicio_button = ft.ElevatedButton("Reiniciar Aplicación", on_click=reiniciar_aplicacion, color=ft.colors.ORANGE)

    formulario_seccion = ft.Column(
        [
            header,
            sueldo_input,
            gasto_input,
            ft.ElevatedButton("Guardar Gasto", on_click=guardar_gasto),
            mensaje,
            total_text,
            deuda_list_text,  # Aquí se agregan las deudas debajo del total restante
            tiene_deudas_checkbox,
            cantidad_deudas_input,
            agregar_deudas_button,
            alerta_button,  # Botón de alerta que aparece si es necesario
            ft.ElevatedButton("Ver plan recomendado", on_click=ver_plan_recomendado),
            reinicio_button  # Botón de reinicio
        ]
    )

    # Layout de la app
    page.add(formulario_seccion, deudas_seccion, plan_seccion, alerta_seccion)

# Ejecutar la aplicación
ft.app(target=main)
