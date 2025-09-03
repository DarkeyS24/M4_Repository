# 📌 Documentação do Sistema - Gráficos, Tabelas e Integração com SQL/EF

Este documento descreve os métodos e trechos de código utilizados para gerar gráficos, tabelas, exportação para Excel e consultas utilizando **Entity Framework 5.0** e **ADO.NET** (`SqlConnection`, `SqlCommand`, `SqlDataReader`).

---

## 📊 Gráficos

### 1. Gráfico de Pizza – Transportes
```csharp
public void SetTransportsPieChart()
{
    var atendimentos = context.TransferenciasPaciente.ToList();
    pieChart.Legends.Clear();
    pieChart.ChartAreas.Clear();
    pieChart.Series.Clear();

    Series series = new Series() { ChartType = SeriesChartType.Pie };
    series.Points.AddXY("Ambulâncias", atendimentos.Count(a => a.TipoTransporte == "Ambulâncias"));
    series.Points.AddXY("UTI Móvel", atendimentos.Count(a => a.TipoTransporte == "UTI Móvel"));
    series.Points.AddXY("Helicóptero", atendimentos.Count(a => a.TipoTransporte == "Helicóptero"));

    pieChart.ChartAreas.Add(new ChartArea());
    pieChart.Series.Add(series);

    valueLbl.Text = atendimentos.Sum(a => a.ValorTotalPago).ToString();
    qtdTranferenciasLbl.Text = atendimentos.Count().ToString();
}
```

---

### 2. Gráfico de Linhas, Barras e Colunas – Atendimentos
```csharp
public void SetAtendimetosChart()
{
    var atendimentos = context.Atendimento.ToList();
    chartAtendimento.Legends.Clear();
    chartAtendimento.ChartAreas.Clear();
    chartAtendimento.Series.Clear();

    Series series = new Series()
    {
        ChartType = ChartAtendimentoCb.SelectedIndex == 0 
            ? SeriesChartType.Line 
            : ChartAtendimentoCb.SelectedIndex == 1 
                ? SeriesChartType.Bar 
                : SeriesChartType.Column
    };

    series.Points.AddXY("Consulta", atendimentos.Count(a => a.TipoAtendimentoId == 1));
    series.Points.AddXY("Cirurgia", atendimentos.Count(a => a.TipoAtendimentoId == 2));
    series.Points.AddXY("Internação", atendimentos.Count(a => a.TipoAtendimentoId == 3));
    series.Points.AddXY("UTI", atendimentos.Count(a => a.TipoAtendimentoId == 4));

    chartAtendimento.ChartAreas.Add(new ChartArea());
    chartAtendimento.Series.Add(series);
}
```

---

## 📅 Tabela com Somatórias de Transferências por Mês
```csharp
public void SetDGV()
{
    var atendimentos = context.TransferenciasPaciente.ToList();
    if (atendimentos.Count > 0)
    {
        janeiroColum.HeaderText = $"01/{year}";
        FevereiroColumn.HeaderText = $"02/{year}";
        // ... demais meses

        var enero = atendimentos.Where(a => a.TipoTransporte == "Ambulância" &&
                                            a.DataTransferencia.Value.Date.Month == 1 &&
                                            a.DataTransferencia.Value.Date.Year == year)
                                .Sum(a => a.ValorTotalPago);

        // Repetição para cada mês e cada tipo de transporte

        dgvSolicitacoes.Rows.Add("Ambulância", enero, febrero, marzo, abril, mayo, junio, 
                                 julio, agosto, septiembre, octrubre, noviembre, diciembre);
        // Repetição para "UTI Móvel" e "Helicóptero"
    }
}
```

---

## 📤 Exportação de Dados para Excel
```csharp
private void exportBtn_Click(object sender, EventArgs e)
{
    excel.Application application = new excel.Application();
    excel.Workbook workbook = application.Workbooks.Add();
    excel.Worksheet worksheet = workbook.Worksheets.Add();

    worksheet.Name = "Filtered Table";
    var columsCount = 0;

    for (int i = 0; i < dgvFiltered.Columns.Count; i++)
    {
        worksheet.Cells[1, i + 1] = dgvFiltered.Columns[i].HeaderText;
        columsCount++;
    }

    var rowCount = 1;
    foreach (DataGridViewRow row in dgvFiltered.Rows)
    {
        if (!row.IsNewRow)
        {
            for (int i = 1; i < columsCount; i++)
            {
                worksheet.Cells[(rowCount + 1), i].Value = row.Cells[i - 1].Value.ToString();
            }
        }
        rowCount++;
    }

    worksheet.Columns.AutoFit();
    string filePath = AppDomain.CurrentDomain.BaseDirectory + $@"Excel\{DateTime.Now:yyyyMMdd_HHmmss}_Filtered_Table";
    workbook.SaveAs(filePath);
    workbook.Close();
    application.Quit();

    MessageBox.Show("Dados Exportados");
}
```

---

## 🧹 Limpeza de DateTimePicker
```csharp
private void limparInicioBtn_Click(object sender, EventArgs e)
{
    inicio = false;
    inicioPicker.Format = DateTimePickerFormat.Custom;
    inicioPicker.CustomFormat = " ";
}

private void terminoPicker_ValueChanged(object sender, EventArgs e)
{
    termino = true;
    terminoPicker.CustomFormat = "dd/MM/yyyy";
}
```

---

## 🔎 Filtro de Tabela (FilterTable)
```csharp
private void FilterTable()
{
    var list = atdList;

    if (inicio)
        list = list.Where(l => l.DataIncioTratamento == inicioPicker.Value).ToList();

    if (termino)
        list = list.Where(l => l.DataTerminoTratamento == terminoPicker.Value).ToList();

    if (!string.IsNullOrEmpty(pacientesTxt.Text))
        list = list.Where(l => l.Paciente.Nome.ToLower().Contains(pacientesTxt.Text.ToLower())).ToList();

    // Filtros por atendimento, sexo, origem, destino
    // Limitação por quantidade (numberPicker)

    if (list.Count == 0)
        MessageBox.Show("Não tem items na lista com esses parametros");
}
```

---

## ⚙️ Consultas com Entity Framework 5.0
Exemplo de consulta direta em entidade relacionada:
```csharp
var list = sessao3.Usuario.ToList();

if (list.Any())
{
    dataGridView1.Rows.Clear();
    foreach (var item in list)
    {
        dataGridView1.Rows.Add(item.Pessoa.Nome.ToString());
    }
}
```

### Relações de Chave Estrangeira no `DbModelBuilder`
```csharp
modelBuilder.Entity<Usuario>()

    .HasRequired(u => u.Pessoa)
    .WithOptional(u => u.Usuario)
    .Map(u => u.MapKey("Id"))
    .WillCascadeOnDelete(true);
```

---

## ⚡ Plano B - ADO.NET (SqlConnection, SqlCommand, SqlDataReader)

### Insert
```csharp
string sql = "INSERT INTO Pessoa (Nome, Email) VALUES (@Nome, @Email)";
using (SqlConnection conn = new SqlConnection(connectionString))
{
    conn.Open();
    using (SqlCommand cmd = new SqlCommand(sql, conn))
    {
        cmd.Parameters.AddWithValue("@Nome", "Maria");
        cmd.Parameters.AddWithValue("@Email", "maria@email.com");
        int linhasAfetadas = cmd.ExecuteNonQuery();
    }
}
```

### Read
```csharp
string sql = "SELECT Id, Nome, Email FROM Pessoa";
using (SqlCommand cmd = new SqlCommand(sql, conn))
using (SqlDataReader reader = cmd.ExecuteReader())
{
    while (reader.Read())
    {
        Console.WriteLine($"ID: {reader["Id"]}, Nome: {reader["Nome"]}, Email: {reader["Email"]}");
    }
}
```

### Update
```csharp
string sql = "UPDATE Pessoa SET Email = @Email WHERE Id = @Id";
using (SqlCommand cmd = new SqlCommand(sql, conn))
{
    cmd.Parameters.AddWithValue("@Email", "novoemail@email.com");
    cmd.Parameters.AddWithValue("@Id", 1);
    int linhasAfetadas = cmd.ExecuteNonQuery();
}
```

### Delete
```csharp
string sql = "DELETE FROM Pessoa WHERE Id = @Id";
using (SqlCommand cmd = new SqlCommand(sql, conn))
{
    cmd.Parameters.AddWithValue("@Id", 1);
    int linhasAfetadas = cmd.ExecuteNonQuery();
}
```

---

# ✅ Conclusão
O sistema utiliza **gráficos interativos**, **tabelas de somatórios**, **filtros dinâmicos** e **exportação para Excel**, permitindo também a integração com banco de dados via **Entity Framework 5.0** e **ADO.NET**.  
