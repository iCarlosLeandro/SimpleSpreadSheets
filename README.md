# SimpleSpreadSheets
Hiii!
SpreadSheets
I made several compromises, but it ended up working 😅
This code works as a co-relationship between a simple table of the value of an X product, with the location of the State in which the purchase was made (In this case, I used the states of Brazil as an example).


import gspread
from google.colab import auth
import pandas as pd
import plotly.express as px

auth.authenticate_user()
from google.auth import default
creds, _ = default()
gc = gspread.authorize(creds)

spreadsheet = gc.open('Evo')
worksheet = spreadsheet.worksheet('Folha1')

values = worksheet.get_all_values()

df = pd.DataFrame(values[1:], columns=values[0])

df['Preço'] = pd.to_numeric(df['Preço'].str.replace('R\$', '').str.replace('.', '').str.replace(',', '.').str.strip(), errors='coerce')

df_agregado = df.groupby('Localizaçao')['Preço'].sum().reset_index()

total_sp = 2200 - 400
df_agregado = pd.concat([df_agregado, pd.DataFrame({'Localizaçao': ['SAO PAULO'], 'Preço': [total_sp]})], ignore_index=True)

df_agregado['Sigla'] = df_agregado['Localizaçao'].str.upper().replace({'SAO PAULO': 'SP', 'SAO PAULO/SP': 'SP'}).str.normalize('NFKD').str.encode('ascii', errors='ignore').str.decode('utf-8').apply(lambda x: x[:2])
df_agregado['Sigla'] = df_agregado['Sigla'].replace({'S': 'SP'})
df_agregado = df_agregado.groupby('Sigla', as_index=False)['Preço'].sum()

siglas_desejadas = ['AC', 'MT', 'SE', 'SC', 'SP', 'RJ', 'PE', 'PA']

df_agregado_filtrado = df_agregado[df_agregado['Sigla'].isin(siglas_desejadas)]

fig = px.bar(df_agregado_filtrado, x='Sigla', y='Preço', labels={'Preço': 'Valor Total'}, title='Preço por Estado (Sigla Agregada)')
fig.update_layout(xaxis=dict(type='category'))
fig.show()
