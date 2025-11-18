# descaracterizacao-nomes
Altera nomes de clientes, embaralhando informações para não serem mais reconhecidos

O objetivo é ler uma tabela de clientes, misturando as informações a fim de que esses dados não possam mais ser reconhecidos e esse banco de dados poder ser usado em apresentações aos clientes com dados fictícios.

```SQL
-- Descaracteriza nomes de clientes, endereços e e-mails para evitar reconhecimento em apresentações
-- Gerar nomes e sobrenomes clientes para depois ofuscar
if object_id('tmp_nome_fake') is not null drop table tmp_nome_fake;

create table tmp_nome_fake
(idcliente bigint primary key,
nome varchar(255),
PrimeiroNome varchar(255),
SegundoNome varchar(255),
TerceiroNome varchar(255),
QuartoNome varchar(255),
sexo varchar(1))
GO

-- Grava na tabela nome fake a partir
-- dos nomes reais em cliente
insert into tmp_nome_fake

Select idcliente,nomeCliente,PrimeiroNome, SegundoNome,
case when len(TerceiroNome) <= 1 then case when len(QuartoNome)<6 then 'Santana' else 'Flaubert' End
	when TerceiroNome = 'DOS' then case when len(QuartoNome)<6 then 'Porto' else 'Humboldt' End
	when TerceiroNome = 'DE' then case when len(QuartoNome)<6 then 'Costa' else 'Silva' End
	when TerceiroNome = 'DA' then case when len(QuartoNome)<6 then 'Miranda'  else 'Souza' End
	when TerceiroNome = 'DAS' then case when len(QuartoNome)<6then 'Lisboa' else 'Hetcher' End
	when TerceiroNome = 'DO' then case when len(QuartoNome)<6 then 'Assunção' else 'Bravia' End
	Else TerceiroNome
	End as TerceiroNome, 
QuartoNome,
Sexo
from
(Select idcliente,nomeCliente,PrimeiroNome,
case when len(SegundoNome) <= 1 then case when len(TerceiroNome)>0 and len(TerceiroNome)<6 then case when (idcliente % 2) = 0 then 'Lima' else 'Yumi' End else TerceiroNome End
	when SegundoNome = 'DOS' then case when len(TerceiroNome)>0 and len(TerceiroNome)<6 then case when (idcliente % 2) = 0 then 'Silvan' else 'Andrade' End else TerceiroNome End
	when SegundoNome = 'DE' then case when len(TerceiroNome)>0 and len(TerceiroNome)<6 then case when (idcliente % 2) = 0 then 'Castro' else 'Roger' End  else TerceiroNome End
	when SegundoNome = 'DA' then case when len(TerceiroNome)>0 and len(TerceiroNome)<6 then case when (idcliente % 2) = 0 then 'Neggen' else 'Bianco' End  else TerceiroNome End
	when SegundoNome = 'DAS' then case when len(TerceiroNome)>0 and len(TerceiroNome)<6 then case when (idcliente % 2) = 0 then 'Tralli' else 'Taggliaferro' End else TerceiroNome End
	when SegundoNome = 'DO' then case when len(TerceiroNome)>0 and len(TerceiroNome)<6 then case when (idcliente % 2) = 0 then 'Dias' else 'Zeist' End else TerceiroNome End
	when len(SegundoNome)<=1 then case when len(TerceiroNome)>0 and len(TerceiroNome)<6 then case when (idcliente % 2) = 0 then 'Delli' else 'Oppenheimer' End else TerceiroNome End	Else SegundoNome
	End as SegundoNome,
TerceiroNome,
case when len(QuartoNome) < 1 then Null
	when QuartoNome = 'DOS' then 'Maia'
	when QuartoNome = 'DE' then case when (idcliente % 2) = 0 then 'Araújo' else 'Kirk' End
	when QuartoNome = 'DA' then case when (idcliente % 2) = 0 then 'Bet' else 'Freixo' End
	when QuartoNome = 'DAS' then case when (idcliente % 2) = 0 then 'Breinch' else 'Machado' End
	when QuartoNome = 'DO'  then case when (idcliente % 2) = 0 then 'Polo' else 'Schneider' End
	when len(QuartoNome) = 1  then case when (idcliente % 2) = 0 then 'Leão' else 'Matos' End
	Else QuartoNome
	End as QuartoNome,
sexo
from
(Select idcliente,nomeCliente, sexo,
ltrim(rtrim(substring(nomeCliente,1,pos1))) as PrimeiroNome,
ltrim(rtrim(substring(nomeCliente,pos1,pos2-pos1))) as SegundoNome,
ltrim(rtrim(substring(nomeCliente,pos2,pos3-pos2))) as TerceiroNome,
ltrim(rtrim(substring(nomeCliente,pos3,pos4-pos3))) as QuartoNome
from (
 select *,
 case when charindex(' ',nomeCliente,Pos3+1) = 0 then len(nomeCliente)+1
 else charindex(' ',nomeCliente,Pos3+1)
 End as Pos4
 from
(select *,
 case when charindex(' ',nomeCliente,Pos2+1) = 0 then len(nomeCliente)+1
 else charindex(' ',nomeCliente,Pos2+1)
 End as Pos3
 from
 (select *,
  case when charindex(' ',nomeCliente,Pos1+1) = 0 then len(nomeCliente)+1
  else charindex(' ',nomeCliente,Pos1+1) 
  End as Pos2
  from
  (select 
   idcliente,nomeCliente,sexo, 
   charindex(' ',nomeCliente) as Pos1
   from
   cliente
   where nomeCliente is not null) a1) 
  a2) a3 where Pos2<=pos3) a4) a5) a6

-- Harmoniza nome com sexo porque na base original existem erros
-- o sexo que for quantidade maior do primeiro nome será considerado o correto
with tmpSexo as
	(select *
	from
	(select idcliente,nomeCliente,sexo,
	substring(a.nomeCliente,1,charindex(' ',nomeCliente)) as primeiro,
	(select top 1 sexo
	from
	tmp_nome_fake tmp
	where
	tmp.PrimeiroNome = substring(a.nomeCliente,1,charindex(' ',nomeCliente))
	group by primeiroNome, sexo
	order by primeiroNome,count(1) desc
	) as novoSexo
	from cliente a) a1
	where
	a1.sexo <> a1.novosexo and
	a1.novosexo is not null)
update cliente
set
sexo = tmp.novoSexo
from cliente a inner join tmpSexo tmp on (a.idcliente = tmp.idcliente)

-- Atualiza a tabela nome fake porque também deve estar errada
with tmpSexo2 as
	(select *
	from
	(select idcliente,PrimeiroNome,sexo,
	SegundoNome,
	(select top 1 sexo
	from
	tmp_nome_fake tmp
	where
	tmp.PrimeiroNome = a.primeiroNome
	group by primeiroNome, sexo
	order by primeiroNome,count(1) desc
	) as novoSexo
	from tmp_nome_fake a) a1
	where
	a1.sexo <> a1.novosexo and
	a1.novosexo is not null)
update tmp_nome_fake
set
sexo = tmp.novoSexo
from tmp_nome_fake a inner join tmpSexo2 tmp on (a.idcliente = tmp.idcliente)

-- Cria uma query com primeiro,segundo e terceiro nomes aleatórios já gerados
with tmp_nome as 
(select idcliente,nomeCliente, sexo,
	case when idcliente%2 = 0  then isnull(Primeiro,PrimeiroAlt)
	else 	isnull(PrimeiroAlt,Primeiro) end as Primeironome,
	case when idcliente%2 = 0 then isnull(Segundo,SegundoAlt)
	else 	isnull(SegundoAlt,Segundo)  end as Segundonome,
	case when idcliente%2 = 0 then isnull(QuartoNome,TerceiroAlt)
	else 	isnull(Terceiro,TerceiroAlt)  end as Terceironome
from
(select idcliente,nomeCliente,sexo,
	isnull(PrimeiroNome,PrimeiroAlt) as primeiro,
	isnull(isnull(SegundoNome,SegundoAlt),QuartoNome) as segundo,
	isnull(TerceiroNome,TerceiroAlt) as terceiro,
	QuartoNome,
	PrimeiroAlt,
	SegundoAlt,
	TerceiroAlt
from
(SELECT idcliente,a.nomeCliente, sexo,
(select top 1 t1.primeironome from tmp_nome_fake t1 where a.idcliente+5<=t1.idcliente and t1.primeironome <> substring(a.nomeCliente,1,charindex(' ',nomeCliente)) and a.sexo=t1.sexo) as PrimeiroNome,
(select top 1 t2.segundonome from tmp_nome_fake t2 where a.idcliente+3<=t2.idcliente) as SegundoNome,
(select top 1 t3.terceironome from tmp_nome_fake t3 where a.idcliente+6<=t3.idcliente) as TerceiroNome,
(select top 1 t4.quartonome from tmp_nome_fake t4 where a.idcliente+4<=t4.idcliente) as QuartoNome,
isnull((select top 1 t4.primeironome from tmp_nome_fake t4 where a.idcliente-2<=t4.idcliente and t4.primeironome <> substring(a.nomeCliente,1,charindex(' ',nomeCliente)) and a.sexo=t4.sexo),case when a.sexo='F' and a.idcliente%2=0 then 'Maria' when a.sexo='F' and a.idcliente%2=1 then 'Beatriz' when a.sexo='M' and a.idcliente%2=1 then 'Pedro' else 'João' End) as PrimeiroAlt,
(select top 1 t5.segundonome from tmp_nome_fake t5 where a.idcliente-4<=t5.idcliente) as SegundoAlt,
(select top 1 t6.terceironome from tmp_nome_fake t6 where a.idcliente-3<=t6.idcliente) as TerceiroAlt
FROM
cliente a) a1) a2)

--Atualiza os nomes e e-mail
update cliente
set 
--cliente.matricula = left(matricula,6) + right(matricula,2) + SUBSTRING(matricula,7,2),
cliente.nomeCliente = upper(PrimeiroNome + ' ' + SegundoNome + ' ' + TerceiroNome),
cliente.mail = lower(PrimeiroNome) + '.' + lower(SegundoNome) + '@r.fakecom.br',
cliente.telefone = replace(replace(replace(replace(a.Telefone,4,7),1,2),9,8),3,0),
cliente.telefone2 = replace(replace(replace(replace(a.Telefone2,5,6),2,1),7,8),3,0),
cliente.telefone3 = replace(replace(replace(replace(a.Telefone3,7,5),3,1),6,8),2,4),
cliente.celular = replace(replace(replace(replace(a.Celular,5,6),3,4),9,7),2,1),
cliente.rg = replace(replace(replace(replace(replace(a.rg,8,5),3,1),9,7),2,4),6,0),
cliente.endereco =
replace(
replace(
replace(
replace(
isnull((select top 1 replace(replace(a1.endereco, 'RUA ', 'Av.'),'Travessa','Praça') from
                 cliente a1 where a.idcliente = a1.idcliente+3), 'Rua Principal'),'1','3'),'0','9'),'5','11'),'4','8'),
cliente.numero = isnull((select top 1 numero from cliente a2 where a2.idcliente = a.idcliente+3), 'S/N'),
cliente.bairro = isnull((select top 1 bairro from cliente a1 where a.idcliente = a1.idcliente+2), 'Centro')
from
cliente a inner join tmp_nome tmp on (a.Idcliente = tmp.idcliente)

--Atualiza CPF somente dos clientes que possuem
EXEC prc_Gerador_CPF_CNPJ

Update cliente
set cpf = (select top 1 nr_documento from tmp_cpf_fake t where t.id = idcliente)
where cpf is not null

-- Checa dados ofuscados
Select idcliente,nomeCliente,endereco,complemento,numero,bairro,idcidade,mail
from cliente
order by idcliente

Select idresponsavelfinanceiroFiador,nomeCliente,endereco,complemento,numero,bairro,idcidade, telefone,mail
from responsavelfinanceiroFiador
order by idresponsavelfinanceiroFiador

Select idprofessor,nomeProfessor,endereco,complemento,numero,bairro,idcidade, telefone,mail
from professor
order by idprofessor
