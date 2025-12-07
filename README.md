import pandas as pd
import numpy as np


# 1. Datos

url = "https://www.datos.gov.co/resource/2uxx-gxp3.json"
df = pd.read_json(url)

print("Total de filas cargadas:", df.shape[0])


# 2. Renombrar columnas principales

df = df.rename(columns={
    "genero": "Genero",
    "nombre_dx": "Diagnostico",
    "ocupacion": "Ocupacion",
})

# 3.  género
def limpiar_genero(x):
    if pd.isna(x):
        return "Sin información"
    x = str(x).strip().lower()
    if x in ["m", "masculino", "hombre"]:
        return "Masculino"
    if x in ["f", "femenino", "mujer"]:
        return "Femenino"
    return "Otro"

df["Genero"] = df["Genero"].apply(limpiar_genero)


# 4. PRIMERA TABLA

columnas_principales = [
    "Genero",
    "Diagnostico",
    "Ocupacion",

]

# Filtrar columnas que existan realmente en el dataset
columnas_principales = [c for c in columnas_principales if c in df.columns]

tabla_principal = df[columnas_principales].sample(20, random_state=40)

print("\n=== TABLA PRINCIPAL: 20 CASOS ALEATORIOS ")
display(tabla_principal)


# 5. Cantidad por género
print("\n=== CANTIDAD DE CASOS POR GÉNERO ===")
display(df["Genero"].value_counts())



# 6. Diagnósticos más frecuentes
print("\n=== DIAGNÓSTICOS MÁS FRECUENTES ===")
display(df["Diagnostico"].value_counts().head(10))


# 7. Promedios por género
for col in ["peso", "talla", "imc", "edad"]:
    if col in df.columns:
        df[col] = pd.to_numeric(df[col], errors='coerce')

df = df.rename(columns={"edad":"Edad", "peso":"Peso", "talla":"Talla", "imc":"IMC"})
prom_cols = [c for c in ["Edad","Peso","Talla","IMC"] if c in df.columns]

print("\n=== PROMEDIOS POR GÉNERO (Edad, Peso, Talla, IMC) ===")
display(df.groupby("Genero")[prom_cols].mean().round(1))


# 8. Riesgo metabólico

def riesgo(imc):
    if pd.isna(imc):
        return "No evaluado"
    if imc >= 30:
        return "Alto"
    if imc >= 25:
        return "Moderado"
    return "Bajo"

if "IMC" in df.columns:
    df["Riesgo_Metabolico"] = df["IMC"].apply(riesgo)

print("\n=== RIESGO METABÓLICO (10 ejemplos aleatorios) ===")
if "Riesgo_Metabolico" in df.columns:
    display(df[["Edad", "IMC", "Riesgo_Metabolico"]].sample(10))

