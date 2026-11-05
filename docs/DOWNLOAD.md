# Como obter os dados

Os dados **não** estão neste repositório (licença CC BY-NC-ND 4.0: imagens não são redistribuídas).

- **Dataset:** Harvard-FairVision, subconjunto DR (Luo et al., arXiv:2310.02492)
- **Metadados (Hugging Face):** `load_dataset("harvardairobotics/FairVision", "dr")`. A coluna `use` define a divisão oficial (training / validation / test).
- **Imagens SLO (JPG):** vêm de um arquivo ZIP hospedado no Google Drive. O notebook (`notebooks/artigo3_dr_fairness.ipynb`, células 3 a 5) lê o ZIP remotamente por requisições com `Range` e salva os JPGs em `/content/slo`.
- **Divisão:** oficial. As listas de IDs e rótulos estão em `data/splits/`.

Os arquivos em `results/predictions/` contêm apenas id, raça, sexo, rótulo, probabilidade e previsão.
