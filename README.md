## **Justificativa das Escolhas da Arquitetura e Custos Estimados**

### 📌 **Visão Geral**
A arquitetura foi projetada para otimizar a ingestão, processamento e armazenamento de dados utilizando componentes escaláveis e eficientes. Com base na análise de volumetria dos eventos, estimamos os custos e configuramos os serviços AWS necessários para garantir um balanceamento entre performance e custo.

---

## **🔍 Justificativa por Serviço**

### **1️⃣ Amazon Simple Queue Service (SQS)**
💰 **Custo Mensal:** **$0,02**  
✅ **Motivo da Escolha:**  
- Utilizado como **fila principal** para desacoplar a entrada de mensagens e permitir um processamento assíncrono eficiente.
- Foi projetado com **fila FIFO**, garantindo a ordem correta das mensagens e eliminando duplicações indesejadas.
- **Custo extremamente baixo**, dado que a volumetria analisada não exige uma grande escala.

🛠 **Configuração Utilizada:**  
- **1 milhão de solicitações/mês**  
- **1GB de entrada e saída de dados/mês**  

---

### **2️⃣ Amazon S3 (DataLake - Armazenamento)**
💰 **Custo Inicial:** **$60,59**  
💰 **Custo Mensal:** **$5,05**  
✅ **Motivo da Escolha:**  
- Idealizado para armazenar **dados brutos antes do processamento** (DataLake).
- Embora não esteja implementado na primeira fase da arquitetura, sua **escalabilidade** e **baixo custo** tornam viável seu uso futuro.
- **Evita custos elevados de armazenamento em banco de dados relacional**.

🛠 **Configuração Utilizada:**  
- **1GB de armazenamento/mês**  
- **1.000.000 de requisições PUT/COPY/LIST**  
- **5000 requisições GET/SELECT**  

---

### **3️⃣ Amazon RDS for PostgreSQL (Data Warehouse + DB Gold)**
💰 **Custo Mensal:** **$89,26**  
✅ **Motivo da Escolha:**  
- Escolhido como **banco de dados estruturado** para armazenar os dados tratados, garantindo **baixo tempo de resposta e consultas otimizadas**.
- Responsável por **armazenar as views processadas pelo segundo componente**, otimizando as consultas do frontend.
- Suporte para **Multi-AZ**, garantindo **alta disponibilidade e redundância**.

🛠 **Configuração Utilizada:**  
- **20GB de armazenamento (SSD gp2)**  
- **Instância db.m1.small**  
- **120 horas/semana de utilização sob demanda**  
- **2GB de armazenamento para backups**  

---

### **4️⃣ AWS Lambda (Processamento Serverless)**
💰 **Custo Mensal:** **$62,29**  
✅ **Motivo da Escolha:**  
- Utilizado para **executar componentes serverless**, como:  
  - **Processamento da fila** (leitura, tratamento e inserção no Data Warehouse).  
  - **Geração de views e verificação da integridade do DB Gold**.  
  - **Atualização de cache diário**.  
- **Alto escalonamento automático**, lidando com a variação de carga da fila detectada nas análises.  
- **Redução de custos** quando comparado a instâncias EC2 sempre ativas.

🛠 **Configuração Utilizada:**  
- **240.000 execuções diárias**  
- **512MB de memória temporária alocada**  
- **Modo de invocação em buffer**  

---

## **📊 Relacionamento entre Arquitetura e Volumetria**
A volumetria analisada nos gráficos de eventos por **minuto, hora e dia** mostrou que:

1️⃣ **Os eventos são distribuídos de maneira desigual** → Picos de até **2500 eventos/hora** foram detectados, exigindo um processamento **elástico**.  
2️⃣ **A demanda é cíclica e pode ser processada em lotes** → Isso justifica a escolha do **AWS Lambda**, pois permite processar eventos conforme a necessidade.  
3️⃣ **Picos extremos foram observados em determinados dias** → Para evitar sobrecarga, o **RDS foi dimensionado** com uma instância pequena e armazenamento SSD escalável.  
4️⃣ **A frequência de eventos varia ao longo do tempo** → A fila SQS foi projetada para manter a **ordem e evitar perda de mensagens**, garantindo confiabilidade.  

---

## **📌 Conclusão**
✅ **Escolhas alinhadas com a demanda** → Utilizamos um modelo **híbrido** (serverless + banco de dados tradicional) para garantir **flexibilidade e escalabilidade**.  
✅ **Custos controlados** → O custo total estimado **não ultrapassa $157/mês**, o que é razoável para uma arquitetura **escalável e confiável**.  
✅ **Preparado para crescimento** → O modelo permite **facilidade de expansão**, seja aumentando a capacidade da fila, do RDS ou adicionando cache no frontend.  

