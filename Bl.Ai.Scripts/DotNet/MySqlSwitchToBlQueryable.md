Use the follow syntaxe to execute `SqlAsQueryable`:

## Example 1:
The code:
```c#

const string SQL = """
    SET NAMES 'utf8';

    SELECT mm.Ait AS `Ait`,
    mm.DATA_BASE AS `BaseDate`,
    mm.Municipio AS `City`,
    mm.CODIGO_ORGAO AS `CodigoOrgao`,
    mm.CLIENTE AS `CustomerId`,
    mm.DATA_INFRACAO AS `Date`,
    d.descricao AS `Description`,
    mm.VALOR_DESCONTO AS `DiscountValue`,
    mm.VENCIMENTO AS `DueDate`,
    fc.IdFrota AS `FrotaId`,
    fc.FrotaName AS `FrotaName`,
    mm.GuiaBanco AS `GuiaNotificacao`,
    mm.LOCAL_INFRACAO AS `Local`,
    mm.DESCONTO_PERDIDO AS `LostValue`,
    IFNULL(m.PAGO, 0) AS `MarkedAsPaid`,
    IFNULL(pa.RECEBIDO, 0) AS `MarkedAsPapel`,
    mm.CODIGO_INFRACAO AS `MultaCode`,
    orgao.ORGAO AS `Orgao`,
    NULL AS `OriginalAit`,
    IF(CHAR_LENGTH(mm.CPF_CNPJ) = 0, NULL, mm.CPF_CNPJ) AS `OwnerDocument`,
    mm.Placa AS `Plate`,
    mm.Renavam AS `Renavam`,
    mm.DATA_PESQUISA AS `SearchedAt`,
    mm.SERVICO AS `Service`,
    mm.HORA_INFRACAO AS `Time`,
    mm.VALOR_DESCONTO AS `Value`,
    e.Marca AS `VehicleBrand`,
    e.Categoria AS `VehicleCategory`,
    e.Tipo AS `VehicleKind`,
    e.AnoFabricacao AS `VehicleManufacturedYear` FROM smartec.tabela_multas AS mm
        LEFT JOIN smartec.tabela_multas_pagas  as m on m.AIP = mm.AIT
        LEFT JOIN smartec.tabela_papel  as pa on pa.AIP = mm.AIT
        LEFT JOIN smartec.de_para_descricao as d ON d.codigo=mm.codigo_infracao
        LEFT JOIN smartec.codigo_orgaos_autuadores as orgao On orgao.codigo = mm.codigo_orgao
        LEFT JOIN veiculo.VeiculoEstatico as e on e.Renavam = mm.Renavam
        LEFT JOIN veiculo.FrotaVehicle AS fv ON fv.Renavam = mm.Renavam AND fv.IdClient = @CustomerId
        LEFT JOIN veiculo.FrotaClient AS fc ON fc.IdFrota = fv.IdFrota
    WHERE (mm.DATA_BASE = @P1002) AND (mm.CLIENTE = @P1003) AND mm.SERVICO IN (@P1000,@P1001)
    ORDER BY mm.DATA_PESQUISA DESC, mm.Ait ASC
    LIMIT 50;
    """;

IQueryable<QueryMultaEmbarcadorModel> query =
    _connection.SqlAsQueryable<QueryMultaEmbarcadorModel>(
    new CommandDefinition(
        SQL,
        parameters: new
        {
            P1002 = '2025-01-01',
            P1003 = 'MULTAS DNIT EMBARCADOR',
            P1000 = 'MULTAS DER SP EMBARCADOR',
            P1001 = 'CLTXXXX',
            CustomerId = 'CLTXXXX'
        },
        transaction: _transaction,
        cancellationToken: default));;

if (frota.Count > 0)
{
    var frotaArray = frota.Select(x => x.ToString()).ToArray();
    query = query.Where(x => frotaArray.Contains(x.FrotaId));
}

return query;

```

Should be trasnformed into:
```c#

string[] embarcadorServices = ["MULTAS DER SP EMBARCADOR", "MULTAS DNIT EMBARCADOR"];

const string SQL = """
    FROM smartec.tabela_multas AS mm
    LEFT JOIN smartec.tabela_multas_pagas  as m on m.AIP = mm.AIT
    LEFT JOIN smartec.tabela_papel  as pa on pa.AIP = mm.AIT
    LEFT JOIN smartec.de_para_descricao as d ON d.codigo=mm.codigo_infracao
    LEFT JOIN smartec.codigo_orgaos_autuadores as orgao On orgao.codigo = mm.codigo_orgao
    LEFT JOIN veiculo.VeiculoEstatico as e on e.Renavam = mm.Renavam
    LEFT JOIN veiculo.FrotaVehicle AS fv ON fv.Renavam = mm.Renavam AND fv.IdClient = @CustomerId
    LEFT JOIN veiculo.FrotaClient AS fc ON fc.IdFrota = fv.IdFrota
    """;

IQueryable<QueryMultaEmbarcadorModel> query =
    _connection.SqlAsQueryable<QueryMultaEmbarcadorModel>(
    new CommandDefinition(
        SQL,
        parameters: new
        {
        },
        transaction: _transaction,
        cancellationToken: default))
        .SetColumnName(e => e.Ait, "mm.Ait")
        .SetColumnName(e => e.BaseDate, "mm.DATA_BASE")
        .SetColumnName(e => e.CustomerId, "mm.CLIENTE")
        .SetColumnName(e => e.MarkedAsPaid, "IFNULL(m.PAGO, 0)")
        .SetColumnName(e => e.MarkedAsPapel, "IFNULL(pa.RECEBIDO, 0)")
        .SetColumnName(e => e.FrotaId, "fc.IdFrota")
        .SetColumnName(e => e.FrotaName, "fc.FrotaName")
        .SetColumnName(e => e.OwnerDocument, "IF(CHAR_LENGTH(mm.CPF_CNPJ) = 0, NULL, mm.CPF_CNPJ)")
        // [...]

        .EnsureAllColumnSet()
        .OrderByDescending(e => e.SearchedAt)
        .ThenBy(e => e.Ait)
        .AddSql(Bl.QueryVisitor.MySql.CommandLocaleRegion.Header, "SET NAMES 'utf8'")
        .Where(x => x.BaseDate == baseDate.ToDateTime())
        .Where(x => x.CustomerId == customerId.ToString())
        .Where(x => embarcadorServices.Contains(x.Service));

if (frota.Count > 0)
{
    var frotaArray = frota.Select(x => x.ToString()).ToArray();
    query = query.Where(x => frotaArray.Contains(x.FrotaId));
}

return query;
```

## Observations
- Switch `{0}`, `{1}` parameters to `CONVERT(@ParamName1 USING latin1)`, `CONVERT(@ParamName2 USING latin1)`

## What should be created
So, do the same for this code:

```c#
// _add_the_code_here
```