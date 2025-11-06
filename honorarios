#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
CALCULADORA DE HONORARIOS PROFESIONALES
Sistema de cálculo de honorarios regulación judicial
"""

import streamlit as st
import pandas as pd
import numpy as np
from datetime import datetime, date
import os
import math

# Configuración de la página
st.set_page_config(
    page_title="Calculadora de Honorarios",
    page_icon="⚖️",
    layout="wide",
    initial_sidebar_state="expanded"
)

# CSS personalizado para replicar el diseño original
st.markdown("""
<style>
    /* Colores principales */
    :root {
        --primary: #2E86AB;
        --secondary: #A23B72;
        --success: #F18F01;
        --info: #C73E1D;
        --light: #F8F9FA;
        --dark: #343A40;
        --highlight-ripte: #E8F5E8;
        --highlight-tasa: #E8F5E8;
    }
    
    /* Ocultar Deploy y menú de 3 puntos */
    button[kind="header"] {
        display: none;
    }
    
    /* Ocultar los 3 puntos verticales */
    [data-testid="stHeader"] svg[viewBox="0 0 16 16"] {
        display: none;
    }
    
    /* Ocultar footer */
    footer {
        display: none;
    }
    
    /* Header personalizado */
    .main-header {
        background-color: #2E86AB;
        padding: 20px;
        border-radius: 10px;
        text-align: center;
        color: white;
        margin-bottom: 30px;
    }
    
    .main-header h1 {
        margin: 0;
        font-size: 28px;
        font-weight: bold;
    }
    
    .main-header h2 {
        margin: 5px 0 0 0;
        font-size: 18px;
        font-weight: normal;
    }
    
    /* Tarjetas de resultados */
    .result-card {
        background-color: #F8F9FA;
        border-left: 4px solid #2E86AB;
        padding: 20px;
        border-radius: 8px;
        margin-bottom: 20px;
    }
    
    .result-card.highlight-ripte {
        background-color: #E8F5E8;
        border-left-color: #28a745;
    }
    
    .result-card.highlight-tasa {
        background-color: #E8F5E8;
        border-left-color: #28a745;
    }
    
    .result-card h3 {
        color: #2E86AB;
        font-size: 16px;
        margin-bottom: 10px;
    }
    
    .result-amount {
        font-size: 32px;
        font-weight: bold;
        color: #343A40;
        margin: 10px 0;
    }
    
    .result-detail {
        font-size: 14px;
        color: #666;
        margin-top: 10px;
    }
    
    /* Alertas */
    .alert-box {
        background-color: #C73E1D;
        color: white;
        padding: 15px;
        border-radius: 8px;
        margin: 20px 0;
    }
    
    .alert-box h4 {
        margin-top: 0;
    }
    
    .alert-success {
        background-color: #28a745;
        color: white;
        padding: 15px;
        border-radius: 8px;
        margin: 20px 0;
    }
    
    .alert-warning {
        background-color: #ffc107;
        color: #343A40;
        padding: 15px;
        border-radius: 8px;
        margin: 20px 0;
        font-weight: bold;
    }
    
    /* Fórmula */
    .formula-box {
        background-color: #e7f3ff;
        border: 1px solid #b3d9ff;
        padding: 15px;
        border-radius: 8px;
        font-family: monospace;
        margin: 20px 0;
    }
    
    /* Botones personalizados */
    .stButton>button {
        background-color: #2E86AB;
        color: white;
        font-weight: bold;
        border-radius: 5px;
        padding: 10px 25px;
        border: none;
    }
    
    .stButton>button:hover {
        background-color: #1a5f7a;
    }
    
    /* Tablas */
    .dataframe {
        font-size: 14px;
    }
    
    /* Sidebar */
    .css-1d391kg {
        background-color: #F8F9FA;
    }
    
    /* Mantener columnas proporcionales */
    [data-testid="stHorizontalBlock"] {
        align-items: flex-start !important;
    }

    /* Tarjetas con alturas coherentes */
    .result-card {
        width: 100% !important;
        min-height: 200px;
        margin-bottom: 18px;
    }
</style>
""", unsafe_allow_html=True)

# Paths de datasets
DATASET_DIR = os.path.abspath(os.path.dirname(__file__))
PATH_JUS = os.path.join(DATASET_DIR, "dataset_jus.csv")

# Funciones auxiliares
def cargar_dataset_jus():
    """Carga el dataset de valores del JUS"""
    try:
        df = pd.read_csv(PATH_JUS)
        df['Fecha'] = pd.to_datetime(df['Fecha'])
        df = df.sort_values('Fecha')
        return df
    except Exception as e:
        st.error(f"Error al cargar dataset_jus.csv: {str(e)}")
        return None

def obtener_valor_jus(fecha, df_jus):
    """Obtiene el valor del JUS vigente para una fecha determinada"""
    if df_jus is None or df_jus.empty:
        return None
    
    # Encontrar el valor vigente (la fecha más reciente anterior o igual a la fecha consultada)
    df_validos = df_jus[df_jus['Fecha'] <= fecha]
    
    if df_validos.empty:
        # Si no hay fechas anteriores, tomar el primer valor disponible
        return df_jus.iloc[0]['Valor_JUS']
    
    return df_validos.iloc[-1]['Valor_JUS']

def format_money(valor):
    """Formatea valores monetarios"""
    return f"$ {valor:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")

# Header principal
st.markdown("""
<div class="main-header">
    <h1>⚖️ CALCULADORA DE HONORARIOS PROFESIONALES</h1>
    <h2>Regulación Judicial - Cálculo de Honorarios y División</h2>
</div>
""", unsafe_allow_html=True)

# Cargar dataset
df_jus = cargar_dataset_jus()

# Sidebar - Inputs
st.sidebar.header("📝 DATOS DE LA REGULACIÓN")

# Selector de función
funcion = st.sidebar.radio(
    "Seleccione la función:",
    ["🔢 FUNCIÓN 1: Cálculo de Honorarios", "📊 FUNCIÓN 2: División de Honorarios (25%)"],
    index=0
)

st.sidebar.markdown("---")

# Variable para controlar el cálculo
calcular = False

# FUNCIÓN 1: CÁLCULO DE HONORARIOS
if funcion == "🔢 FUNCIÓN 1: Cálculo de Honorarios":
    st.sidebar.subheader("Parámetros de Cálculo")
    
    # Fecha de la base regulatoria
    fecha_base = st.sidebar.date_input(
        "Fecha de la Base Regulatoria",
        value=date.today(),
        help="Fecha sobre la cual se realizará el cálculo"
    )
    
    # Monto total de la base regulatoria
    monto_base = st.sidebar.number_input(
        "Monto Total de la Base Regulatoria ($)",
        min_value=0.00,
        value=1000000.00,
        step=1000.00,
        format="%.2f",
        help="Monto en pesos sobre el cual se calculan los honorarios"
    )
    
    # Porcentaje a aplicar
    porcentaje = st.sidebar.slider(
        "Porcentaje a Aplicar (%)",
        min_value=7.5,
        max_value=25.0,
        value=15.0,
        step=0.5,
        help="Porcentaje de honorarios según regulación"
    )
    
    st.sidebar.markdown("---")
    calcular = st.sidebar.button("🔍 CALCULAR HONORARIOS", use_container_width=True)
    
    # Realizar cálculo si se presiona el botón
    if calcular:
        if df_jus is not None:
            # Obtener valor del JUS vigente
            valor_jus = obtener_valor_jus(pd.Timestamp(fecha_base), df_jus)
            
            if valor_jus:
                # Calcular honorarios en pesos
                honorarios_pesos = monto_base * (porcentaje / 100)
                
                # Convertir a JUS
                honorarios_jus = honorarios_pesos / valor_jus
                
                # Mínimo de 7 JUS
                minimo_jus = 7
                minimo_pesos = minimo_jus * valor_jus
                aplica_minimo = honorarios_jus < minimo_jus
                
                # Resultados finales
                honorarios_final_jus = max(honorarios_jus, minimo_jus)
                honorarios_final_pesos = honorarios_final_jus * valor_jus
                
                # Mostrar resultados
                st.markdown("## 💰 RESULTADOS DEL CÁLCULO")
                
                col1, col2, col3 = st.columns(3)
                
                with col1:
                    st.markdown(f"""
                    <div class="result-card">
                        <h3>📅 Fecha Base</h3>
                        <div class="result-amount">{fecha_base.strftime('%d/%m/%Y')}</div>
                        <div class="result-detail">Valor JUS vigente: {format_money(valor_jus)}</div>
                    </div>
                    """, unsafe_allow_html=True)
                
                with col2:
                    st.markdown(f"""
                    <div class="result-card">
                        <h3>💵 Base Regulatoria</h3>
                        <div class="result-amount">{format_money(monto_base)}</div>
                        <div class="result-detail">Porcentaje aplicado: {porcentaje}%</div>
                    </div>
                    """, unsafe_allow_html=True)
                
                with col3:
                    st.markdown(f"""
                    <div class="result-card">
                        <h3>⚖️ Honorarios Calculados</h3>
                        <div class="result-amount">{format_money(honorarios_pesos)}</div>
                        <div class="result-detail">Equivalente: {honorarios_jus:.2f} JUS</div>
                    </div>
                    """, unsafe_allow_html=True)
                
                # Alerta de mínimo si aplica
                if aplica_minimo:
                    st.markdown(f"""
                    <div class="alert-warning">
                        <h4>⚠️ SE APLICA MÍNIMO DE 7 JUS</h4>
                        <p>El cálculo original ({honorarios_jus:.2f} JUS) es inferior al mínimo legal de 7 JUS.</p>
                        <p><strong>Se aplica el mínimo: {format_money(minimo_pesos)}</strong></p>
                    </div>
                    """, unsafe_allow_html=True)
                
                # Resultado final destacado
                st.markdown("---")
                col_final1, col_final2 = st.columns(2)
                
                with col_final1:
                    st.markdown(f"""
                    <div class="result-card highlight-ripte">
                        <h3>💼 HONORARIOS FINALES EN PESOS</h3>
                        <div class="result-amount" style="color: #28a745;">{format_money(honorarios_final_pesos)}</div>
                        <div class="result-detail">Monto a regular en pesos argentinos</div>
                    </div>
                    """, unsafe_allow_html=True)
                
                with col_final2:
                    st.markdown(f"""
                    <div class="result-card highlight-tasa">
                        <h3>⚖️ HONORARIOS FINALES EN JUS</h3>
                        <div class="result-amount" style="color: #28a745;">{honorarios_final_jus:.2f} JUS</div>
                        <div class="result-detail">Equivalente en JUS (valor: {format_money(valor_jus)})</div>
                    </div>
                    """, unsafe_allow_html=True)
                
                # Detalle del cálculo
                st.markdown("---")
                st.markdown("### 📊 DETALLE DEL CÁLCULO")
                
                st.markdown(f"""
                <div class="formula-box">
                    <p><strong>Base Regulatoria:</strong> {format_money(monto_base)}</p>
                    <p><strong>Porcentaje aplicado:</strong> {porcentaje}%</p>
                    <p><strong>Honorarios = Base × Porcentaje:</strong> {format_money(monto_base)} × {porcentaje}% = {format_money(honorarios_pesos)}</p>
                    <p><strong>Valor JUS vigente al {fecha_base.strftime('%d/%m/%Y')}:</strong> {format_money(valor_jus)}</p>
                    <p><strong>Conversión a JUS:</strong> {format_money(honorarios_pesos)} ÷ {format_money(valor_jus)} = {honorarios_jus:.2f} JUS</p>
                    {f'<p style="color: #856404;"><strong>⚠️ Se aplica mínimo legal: 7 JUS = {format_money(minimo_pesos)}</strong></p>' if aplica_minimo else ''}
                    <p style="margin-top: 15px; font-size: 16px;"><strong>RESULTADO FINAL: {format_money(honorarios_final_pesos)} ({honorarios_final_jus:.2f} JUS)</strong></p>
                </div>
                """, unsafe_allow_html=True)
            else:
                st.error("❌ No se pudo obtener el valor del JUS para la fecha seleccionada")
        else:
            st.error("❌ No se pudo cargar el dataset de valores JUS")

# FUNCIÓN 2: DIVISIÓN DE HONORARIOS (25%)
elif funcion == "📊 FUNCIÓN 2: División de Honorarios (25%)":
    st.sidebar.subheader("Parámetros de División")
    
    # Monto total de la base regulatoria
    monto_base_div = st.sidebar.number_input(
        "Monto Total de la Base Regulatoria ($)",
        min_value=0.00,
        value=1000000.00,
        step=1000.00,
        format="%.2f",
        help="Monto total sobre el cual se divide el 25%"
    )
    
    # Calcular el 25% máximo
    monto_25_pct = monto_base_div * 0.25
    
    st.sidebar.info(f"💡 Monto máximo a distribuir (25%): {format_money(monto_25_pct)}")
    
    st.sidebar.markdown("---")
    st.sidebar.subheader("Representación Letrada")
    
    # Porcentaje representación letrada
    porc_letrado = st.sidebar.slider(
        "Porcentaje Representación Letrada (%)",
        min_value=7.5,
        max_value=25.0,
        value=15.0,
        step=0.5,
        help="Porcentaje base para representación letrada (antes de auxiliares)"
    )
    
    st.sidebar.markdown("---")
    st.sidebar.subheader("Auxiliares de Justicia")
    
    # Auxiliares (máximo 3)
    auxiliar_1 = st.sidebar.slider(
        "Auxiliar 1 (%)",
        min_value=0.0,
        max_value=15.0,
        value=0.0,
        step=0.5,
        help="Porcentaje para primer auxiliar"
    )
    
    auxiliar_2 = st.sidebar.slider(
        "Auxiliar 2 (%)",
        min_value=0.0,
        max_value=15.0,
        value=0.0,
        step=0.5,
        help="Porcentaje para segundo auxiliar"
    )
    
    auxiliar_3 = st.sidebar.slider(
        "Auxiliar 3 (%)",
        min_value=0.0,
        max_value=15.0,
        value=0.0,
        step=0.5,
        help="Porcentaje para tercer auxiliar"
    )
    
    st.sidebar.markdown("---")
    calcular = st.sidebar.button("🔍 CALCULAR DIVISIÓN", use_container_width=True)
    
    # Realizar cálculo si se presiona el botón
    if calcular:
        # Calcular porcentaje neto letrado (después de descontar auxiliares)
        total_auxiliares = auxiliar_1 + auxiliar_2 + auxiliar_3
        porc_letrado_neto = porc_letrado - total_auxiliares
        
        # Validar que no sea negativo
        if porc_letrado_neto < 0:
            st.error(f"❌ ERROR: El porcentaje de auxiliares ({total_auxiliares}%) excede el porcentaje de representación letrada ({porc_letrado}%)")
        else:
            # Calcular montos base (sin IVA ni Caja)
            monto_letrado_base = monto_base_div * (porc_letrado_neto / 100)
            monto_aux_1_base = monto_base_div * (auxiliar_1 / 100)
            monto_aux_2_base = monto_base_div * (auxiliar_2 / 100)
            monto_aux_3_base = monto_base_div * (auxiliar_3 / 100)
            
            # Calcular Caja (10%) e IVA (21%) solo para letrado
            caja_letrado = monto_letrado_base * 0.10
            iva_letrado = monto_letrado_base * 0.21
            
            # Total letrado con Caja e IVA
            monto_letrado_total = monto_letrado_base + caja_letrado + iva_letrado
            
            # Calcular porcentaje efectivo usado del 25%
            total_porcentaje_usado = porc_letrado + total_auxiliares
            
            # Verificar si excede el 25%
            excede_25 = total_porcentaje_usado > 25
            
            # Calcular total general
            total_general = monto_letrado_total + monto_aux_1_base + monto_aux_2_base + monto_aux_3_base
            
            # Mostrar resultados
            st.markdown("## 📊 DIVISIÓN DE HONORARIOS")
            
            # Resumen general
            col1, col2, col3 = st.columns(3)
            
            with col1:
                st.markdown(f"""
                <div class="result-card">
                    <h3>💵 Base Regulatoria</h3>
                    <div class="result-amount">{format_money(monto_base_div)}</div>
                    <div class="result-detail">Monto total del caso</div>
                </div>
                """, unsafe_allow_html=True)
            
            with col2:
                st.markdown(f"""
                <div class="result-card">
                    <h3>📈 Máximo Distribuible (25%)</h3>
                    <div class="result-amount">{format_money(monto_25_pct)}</div>
                    <div class="result-detail">Límite legal parte perdedora</div>
                </div>
                """, unsafe_allow_html=True)
            
            with col3:
                color_porcentaje = "#dc3545" if excede_25 else "#28a745"
                st.markdown(f"""
                <div class="result-card">
                    <h3>📊 Porcentaje Usado</h3>
                    <div class="result-amount" style="color: {color_porcentaje};">{total_porcentaje_usado:.2f}%</div>
                    <div class="result-detail">{'⚠️ EXCEDE EL 25%' if excede_25 else '✅ Dentro del límite'}</div>
                </div>
                """, unsafe_allow_html=True)
            
            # Alerta si excede el 25%
            if excede_25:
                st.markdown(f"""
                <div class="alert-box">
                    <h4>⚠️ ADVERTENCIA: SE EXCEDE EL LÍMITE DEL 25%</h4>
                    <p>El porcentaje total ({total_porcentaje_usado:.2f}%) excede el máximo legal del 25% de la base regulatoria.</p>
                    <p><strong>Debe ajustar los porcentajes de representación letrada y/o auxiliares.</strong></p>
                </div>
                """, unsafe_allow_html=True)
            
            st.markdown("---")
            
            # Detalle de Representación Letrada
            st.markdown("### 👨‍⚖️ REPRESENTACIÓN LETRADA")
            
            col_let1, col_let2 = st.columns(2)
            
            with col_let1:
                st.markdown(f"""
                <div class="result-card highlight-ripte">
                    <h3>💼 Honorarios Base Letrado</h3>
                    <div class="result-amount" style="color: #28a745;">{format_money(monto_letrado_base)}</div>
                    <div class="result-detail">Porcentaje neto: {porc_letrado_neto:.2f}% (después de auxiliares)</div>
                </div>
                """, unsafe_allow_html=True)
            
            with col_let2:
                st.markdown(f"""
                <div class="result-card highlight-tasa">
                    <h3>💰 Total con Caja + IVA</h3>
                    <div class="result-amount" style="color: #28a745;">{format_money(monto_letrado_total)}</div>
                    <div class="result-detail">Incluye Caja (10%) + IVA (21%)</div>
                </div>
                """, unsafe_allow_html=True)
            
            # Detalle del cálculo letrado
            st.markdown(f"""
            <div class="formula-box">
                <p><strong>Porcentaje inicial:</strong> {porc_letrado:.2f}%</p>
                <p><strong>Descuento auxiliares:</strong> -{total_auxiliares:.2f}%</p>
                <p><strong>Porcentaje neto letrado:</strong> {porc_letrado_neto:.2f}%</p>
                <hr>
                <p><strong>Honorarios base:</strong> {format_money(monto_base_div)} × {porc_letrado_neto:.2f}% = {format_money(monto_letrado_base)}</p>
                <p><strong>Caja de Abogados (10%):</strong> {format_money(caja_letrado)}</p>
                <p><strong>IVA (21%):</strong> {format_money(iva_letrado)}</p>
                <hr>
                <p style="font-size: 16px;"><strong>TOTAL LETRADO: {format_money(monto_letrado_total)}</strong></p>
            </div>
            """, unsafe_allow_html=True)
            
            # Detalle de Auxiliares
            if total_auxiliares > 0:
                st.markdown("---")
                st.markdown("### 🔧 AUXILIARES DE JUSTICIA")
                
                cols_aux = st.columns(3)
                
                # Auxiliar 1
                if auxiliar_1 > 0:
                    with cols_aux[0]:
                        st.markdown(f"""
                        <div class="result-card">
                            <h3>👤 Auxiliar 1</h3>
                            <div class="result-amount">{format_money(monto_aux_1_base)}</div>
                            <div class="result-detail">Porcentaje: {auxiliar_1:.2f}%</div>
                        </div>
                        """, unsafe_allow_html=True)
                
                # Auxiliar 2
                if auxiliar_2 > 0:
                    with cols_aux[1]:
                        st.markdown(f"""
                        <div class="result-card">
                            <h3>👤 Auxiliar 2</h3>
                            <div class="result-amount">{format_money(monto_aux_2_base)}</div>
                            <div class="result-detail">Porcentaje: {auxiliar_2:.2f}%</div>
                        </div>
                        """, unsafe_allow_html=True)
                
                # Auxiliar 3
                if auxiliar_3 > 0:
                    with cols_aux[2]:
                        st.markdown(f"""
                        <div class="result-card">
                            <h3>👤 Auxiliar 3</h3>
                            <div class="result-amount">{format_money(monto_aux_3_base)}</div>
                            <div class="result-detail">Porcentaje: {auxiliar_3:.2f}%</div>
                        </div>
                        """, unsafe_allow_html=True)
            
            # Total General
            st.markdown("---")
            st.markdown("### 💼 TOTAL GENERAL")
            
            col_total1, col_total2 = st.columns(2)
            
            with col_total1:
                st.markdown(f"""
                <div class="result-card highlight-ripte">
                    <h3>💰 TOTAL A PAGAR</h3>
                    <div class="result-amount" style="font-size: 36px; color: #28a745;">{format_money(total_general)}</div>
                    <div class="result-detail">Suma de todos los conceptos</div>
                </div>
                """, unsafe_allow_html=True)
            
            with col_total2:
                porcentaje_efectivo = (total_general / monto_base_div) * 100
                st.markdown(f"""
                <div class="result-card highlight-tasa">
                    <h3>📊 Porcentaje Efectivo Total</h3>
                    <div class="result-amount" style="font-size: 36px; color: #28a745;">{porcentaje_efectivo:.2f}%</div>
                    <div class="result-detail">Sobre la base regulatoria</div>
                </div>
                """, unsafe_allow_html=True)
            
            # Tabla resumen
            st.markdown("---")
            st.markdown("### 📋 RESUMEN DETALLADO")
            
            # Crear DataFrame para mostrar
            data_resumen = {
                'Concepto': [],
                'Porcentaje (%)': [],
                'Monto ($)': []
            }
            
            data_resumen['Concepto'].append('Representación Letrada (base)')
            data_resumen['Porcentaje (%)'].append(f"{porc_letrado_neto:.2f}%")
            data_resumen['Monto ($)'].append(format_money(monto_letrado_base))
            
            data_resumen['Concepto'].append('Caja de Abogados (10%)')
            data_resumen['Porcentaje (%)'].append('10.00%')
            data_resumen['Monto ($)'].append(format_money(caja_letrado))
            
            data_resumen['Concepto'].append('IVA (21%)')
            data_resumen['Porcentaje (%)'].append('21.00%')
            data_resumen['Monto ($)'].append(format_money(iva_letrado))
            
            data_resumen['Concepto'].append('SUBTOTAL LETRADO')
            data_resumen['Porcentaje (%)'].append('-')
            data_resumen['Monto ($)'].append(format_money(monto_letrado_total))
            
            if auxiliar_1 > 0:
                data_resumen['Concepto'].append('Auxiliar 1')
                data_resumen['Porcentaje (%)'].append(f"{auxiliar_1:.2f}%")
                data_resumen['Monto ($)'].append(format_money(monto_aux_1_base))
            
            if auxiliar_2 > 0:
                data_resumen['Concepto'].append('Auxiliar 2')
                data_resumen['Porcentaje (%)'].append(f"{auxiliar_2:.2f}%")
                data_resumen['Monto ($)'].append(format_money(monto_aux_2_base))
            
            if auxiliar_3 > 0:
                data_resumen['Concepto'].append('Auxiliar 3')
                data_resumen['Porcentaje (%)'].append(f"{auxiliar_3:.2f}%")
                data_resumen['Monto ($)'].append(format_money(monto_aux_3_base))
            
            data_resumen['Concepto'].append('TOTAL GENERAL')
            data_resumen['Porcentaje (%)'].append(f"{porcentaje_efectivo:.2f}%")
            data_resumen['Monto ($)'].append(format_money(total_general))
            
            df_resumen = pd.DataFrame(data_resumen)
            st.dataframe(df_resumen, use_container_width=True, hide_index=True)

# Información inicial si no se ha calculado
if not calcular:
    st.info("👈 Complete los datos en el panel lateral y presione el botón de calcular para obtener los resultados")
    
    # Mostrar información general
    col1, col2 = st.columns(2)
    
    with col1:
        st.markdown("""
        ### 🔢 Función 1: Cálculo de Honorarios
        - Cálculo basado en monto de base regulatoria
        - Conversión automática a JUS
        - Aplicación de mínimo legal (7 JUS)
        - Valor JUS según fecha de base regulatoria
        """)
    
    with col2:
        st.markdown("""
        ### 📊 Función 2: División de Honorarios
        - División del 25% máximo parte perdedora
        - Representación letrada con Caja (10%) e IVA (21%)
        - Hasta 3 auxiliares de justicia
        - Control automático del límite del 25%
        """)

# Footer
st.markdown("---")
st.markdown("""
<div style='text-align: center; color: #666; padding: 20px;'>
    <p><strong>Calculadora de Honorarios Profesionales</strong><br>
    Sistema de Regulación Judicial<br>
    Versión 1.0 - Los cálculos deben ser verificados manualmente</p>
</div>
""", unsafe_allow_html=True)
