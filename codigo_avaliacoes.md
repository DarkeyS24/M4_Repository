``` csharp
private void SetJSON(AvaliacaoRisco avaliacao) 
{ 
    var basePath = AppDomain.CurrentDomain.BaseDirectory; 
    var caminho = Path.Combine(basePath,"AvaliacoesExcluidas.json"); 
    if (File.Exists(caminho)) 
    { 
        var content = File.ReadAllText(caminho); 
        if (!string.IsNullOrWhiteSpace(content)) 
        { 
            var avaliacoes = JsonConvert.DeserializeObject<List<AvaliacaoRisco>>(content); 
            avaliacoes.Add(avaliacao); 
            var json = JsonConvert.SerializeObject(avaliacoes); 
            File.WriteAllText(caminho, json); 
        }
        else 
        { 
            List<AvaliacaoRisco> avaliacaos = new List<AvaliacaoRisco>(); 
            avaliacaos.Add(avaliacao); 
            var json = JsonConvert.SerializeObject(avaliacaos); 
            File.WriteAllText(caminho, json); 
        } 
    } 
}

private void chart1_MouseClick(object sender, MouseEventArgs e) 
{ 
    HitTestResult result = chart1.HitTest(e.X, e.Y); 
    if (result.ChartElementType == ChartElementType.DataPoint) 
    { 
        DataPoint point = result.Series.Points[result.PointIndex]; 
        nomeTipo = point.LegendText; 
        SetList(); 
    } 
}

private void dgvAvalições_CellContentClick(object sender, DataGridViewCellEventArgs e) 
{ 
    if (e.ColumnIndex == 6) 
    { 
        // Detalhes 
    }
    else if (e.ColumnIndex == 7) 
    { 
        // Excluir 
        if (dgvAvalições.SelectedRows.Count == 1) 
        { 
            var res = MessageBox.Show("Deseja Excluir essa avaliação", "Confirmar Exclusão", MessageBoxButtons.YesNo); 
            if (res == DialogResult.Yes) 
            { 
                if (!dgvAvalições.Rows[e.RowIndex].IsNewRow) 
                { 
                    var row = dgvAvalições.Rows[e.RowIndex]; 
                    var avaliacao = context.AvaliacaoRisco.Find(int.Parse(row.Cells[0].Value.ToString())); 
                    SetJSON(avaliacao); 
                    context.AvaliacaoRisco.Remove(avaliacao); 
                    context.SaveChanges(); 
                    SetList(); 
                } 
            } 
        }
        else if (dgvAvalições.SelectedRows.Count > 1) 
        { 
            var res = MessageBox.Show($"Deseja Excluir estas {dgvAvalições.SelectedRows.Count} avaliações", "Confirmar Exclusão", MessageBoxButtons.YesNo); 
            if (res == DialogResult.Yes) 
            { 
                if (listaDeletados.Any()) 
                { 
                    context.AvaliacaoRisco.RemoveRange(listaDeletados); 
                    context.SaveChanges(); 
                } 
                listaDeletados = new List<AvaliacaoRisco>(); 
                foreach (DataGridViewRow item in dgvAvalições.SelectedRows) 
                { 
                    if (!item.IsNewRow) 
                    { 
                        var avaliacao = context.AvaliacaoRisco.Find(int.Parse(item.Cells[0].Value.ToString())); 
                        SetJSON(avaliacao); 
                        listaDeletados.Add(avaliacao); 
                        SetList(); 
                    } 
                } 
                List<DataGridViewRow> list = new List<DataGridViewRow>(); 
                foreach (var item in listaDeletados) 
                { 
                    foreach (DataGridViewRow item1 in dgvAvalições.Rows) 
                    { 
                        var id = int.Parse(item1.Cells[0].Value.ToString()); 
                        if (id == item.Id) 
                        { 
                            list.Add(item1); 
                        } 
                    } 
                } 
                if (list.Any()) 
                { 
                    foreach (var item in list) 
                    { 
                        dgvAvalições.Rows.Remove(item); 
                    } 
                } 
            } 
        } 
        else 
        { 
            MessageBox.Show("Selecione uma avalição", "Avalições", MessageBoxButtons.OK); 
        } 
    } 
}

public void SetList() 
{ 
    dgvAvalições.Rows.Clear(); 
    context.AvaliacaoRisco.ToList(); 
    if (!string.IsNullOrEmpty(textBox1.Text)) 
    { 
        var atendimentos = string.IsNullOrEmpty(nomeTipo) ? context.AtendimentoProduto 
            .Include(ap => ap.Atendimento) 
            .ThenInclude(a => a.Cliente) 
            .ThenInclude(c => c.IdNavigation) 
            .ThenInclude(id => id.Endereco) 
            .Include(ap => ap.Produto) 
            .Where(ap => ap.Produto.Nome.Contains(textBox1.Text)).ToList() : 
            context.AtendimentoProduto 
            .Include(ap => ap.Atendimento) 
            .ThenInclude(a => a.Cliente) 
            .ThenInclude(c => c.IdNavigation) 
            .ThenInclude(id => id.Endereco) 
            .Include(ap => ap.Produto) 
            .Where(ap => ap.Produto.Nome.Contains(textBox1.Text) && ap.Produto.Tipo == nomeTipo).ToList(); 

        if (atendimentos.Any()) 
        { 
            List<AvaliacaoRisco> list = new List<AvaliacaoRisco>(); 
            dgvAvalições.Rows.Clear(); 
            List<AvaliacaoRisco> temp = new List<AvaliacaoRisco>(); 
            var avaliacoes = context.AvaliacaoRisco 
                .Include(a => a.Profissional) 
                .ThenInclude(p => p.IdNavigation) 
                .ThenInclude(e => e.Endereco) 
                .Include(a => a.Cliente) 
                .ThenInclude(p => p.IdNavigation) 
                .ThenInclude(e => e.Endereco) 
                .ToList(); 
            if (!string.IsNullOrEmpty(nomeTipo)) 
            { 
                foreach (var item in atendimentos) 
                { 
                    foreach (var item1 in avaliacoes) 
                    { 
                        if(item1.ClienteId == item.Atendimento.ClienteId) 
                        { 
                            temp.Add(item1); 
                        } 
                    } 
                } 
                avaliacoes = temp; 
            } 
            foreach (var item in atendimentos) 
            { 
                foreach (var item1 in avaliacoes) 
                { 
                    if (item1.ClienteId == item.Atendimento.ClienteId) 
                    { 
                        list.Add(item1); 
                    } 
                } 
            } 
            if (list.Any()) 
            { 
                foreach (var item in list) 
                { 
                    dgvAvalições.Rows.Add(item.Id, item.DataAvaliacao.ToString("dd/MM/yyyy"), item.Cliente.IdNavigation.Nome, item.Profissional.IdNavigation.Nome, item.NotaFinalPonderada, item.NivelRisco); 
                } 
                chart1.ChartAreas.Clear(); 
                chart1.Series.Clear(); 
                chart1.Legends.Clear(); 
                chart1.ChartAreas.Add(new ChartArea()); 
                Series serie = new Series() { ChartType = SeriesChartType.Pie}; 
                var listaFiltrada = atendimentos.GroupBy(l => l.Produto.Tipo).Select(l => new { tipo = l.Key, value = l.Count()}).ToList(); 
                if (listaFiltrada.Any()) 
                { 
                    medLbl.Text = "0.00%"; 
                    equipLbl.Text = "0.00%"; 
                    foreach (var item in listaFiltrada) 
                    { 
                        switch (item.tipo) 
                        { 
                            case "Medicamento": 
                                medLbl.Text = item.value / listaFiltrada.Sum(l => l.value) * 100 + "%"; 
                                break; 
                            case "Equipamento": 
                                equipLbl.Text = item.value / listaFiltrada.Sum(l => l.value) * 100 + "%"; 
                                break; 
                        } 
                        DataPoint data = new DataPoint(); 
                        serie.Points.AddXY(item.tipo, item.value); 
                        serie.LegendText = item.tipo; 
                    } 
                    chart1.Series.Add(serie); 
                } 
                var enderecos = context.Endereco.ToList(); 
                foreach (var item in avaliacoes) 
                { 
                    enderecos = enderecos.Where(e => e.PessoaId == item.Cliente.IdNavigation.Id).ToList(); 
                } 
                flowLayoutPanel1.Controls.Clear(); 
                var cidades = enderecos.GroupBy(c => c.Cidade).Select(e => new { cidade = e.Key, value = e.Count() }).ToList(); 
                foreach (var item in cidades) 
                { 
                    var perc = (item.value / cidades.Sum(c => c.value)) * 1.0; 
                    var cor = Color.FromArgb((int)(perc * 255), Color.Yellow); 
                    CidadeItem cidade = new CidadeItem(); 
                    cidade.SetData(cor, item.cidade); 
                    flowLayoutPanel1.Controls.Add(cidade); 
                } 
            } 
        } 
    } 
    else 
    { 
        var atendimentos = string.IsNullOrEmpty(nomeTipo) ? context.AtendimentoProduto 
            .Include(ap => ap.Atendimento) 
            .ThenInclude(a => a.Cliente) 
            .ThenInclude(c => c.IdNavigation) 
            .ThenInclude(id => id.Endereco) 
            .Include(ap => ap.Produto) 
            .ToList() : 
            context.AtendimentoProduto 
            .Include(ap => ap.Atendimento) 
            .ThenInclude(a => a.Cliente) 
            .ThenInclude(c => c.IdNavigation) 
            .ThenInclude(id => id.Endereco) 
            .Include(ap => ap.Produto) 
            .Where(ap => ap.Produto.Tipo == nomeTipo).ToList(); 

        List<AvaliacaoRisco> temp = new List<AvaliacaoRisco>(); 
        var avaliacoes = context.AvaliacaoRisco 
            .Include(a => a.Profissional) 
            .ThenInclude(p => p.IdNavigation) 
            .ThenInclude(e => e.Endereco) 
            .Include(a => a.Cliente) 
            .ThenInclude(p => p.IdNavigation) 
            .ThenInclude(e => e.Endereco) 
            .ToList(); 
        if (!string.IsNullOrEmpty(nomeTipo)) 
        { 
            foreach (var item in atendimentos) 
            { 
                foreach (var item1 in avaliacoes) 
                { 
                    if (item1.ClienteId == item.Atendimento.ClienteId) 
                    { 
                        temp.Add(item1); 
                    } 
                } 
            } 
            avaliacoes = temp; 
        } 
        if (avaliacoes.Any()) 
        { 
            foreach (var item in avaliacoes) 
            { 
                dgvAvalições.Rows.Add(item.Id, item.DataAvaliacao.ToString("dd/MM/yyyy"), item.Cliente.IdNavigation.Nome, item.Profissional.IdNavigation.Nome, item.NotaFinalPonderada, item.NivelRisco); 
            } 
        } 
        chart1.ChartAreas.Clear(); 
        chart1.Series.Clear(); 
        chart1.Legends.Clear(); 
        chart1.ChartAreas.Add(new ChartArea()); 
        Series serie = new Series() { ChartType = SeriesChartType.Pie }; 
        var listaFiltrada = atendimentos.GroupBy(l => l.Produto.Tipo).Select(l => new { tipo = l.Key, value = l.Count() }).ToList(); 
        if (listaFiltrada.Any()) 
        { 
            medLbl.Text = "0.00%"; 
            equipLbl.Text = "0.00%"; 
            foreach (var item in listaFiltrada) 
            { 
                switch (item.tipo) 
                { 
                    case "Medicamento": 
                        var valueM = (decimal)item.value / listaFiltrada.Sum(l => l.value) * 100; 
                        medLbl.Text = valueM.ToString("F2") + "%"; 
                        break; 
                    case "Equipamento": 
                        var valueE = (decimal)item.value / listaFiltrada.Sum(l => l.value) * 100; 
                        equipLbl.Text = valueE.ToString("F2") + "%"; 
                        break; 
                } 
                DataPoint data = new DataPoint(); 
                serie.Points.AddXY(item.tipo, item.value); 
                serie.LegendText = item.tipo; 
            } 
            chart1.Series.Add(serie); 
        } 
        var enderecos = context.Endereco.ToList(); 
        var listaEnderecos = new List<Endereco>(); 
        foreach (var item in avaliacoes) 
        { 
            listaEnderecos.AddRange(enderecos.Where(e => e.PessoaId == item.ClienteId).ToList()); 
        } 
        flowLayoutPanel1.Controls.Clear(); 
        var cidades = listaEnderecos.GroupBy(c => c.Cidade).Select(e => new { cidade = e.Key, value = e.Count() }).ToList(); 
        foreach (var item in cidades.OrderByDescending(c => c.value)) 
        { 
            var perc = (decimal)item.value / (decimal)cidades.Sum(c => c.value) * 10M; 
            var cor = Color.FromArgb((int)(perc * 255 > 255 ? 255 : perc * 255), Color.Yellow); 
            CidadeItem cidade = new CidadeItem(); 
            cidade.SetData(cor, item.cidade); 
            flowLayoutPanel1.Controls.Add(cidade); 
        } 
    } 
}
```
