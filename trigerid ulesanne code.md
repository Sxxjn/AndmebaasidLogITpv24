```sql
create database kontserdidRuslan
use kontserdidRuslan

--tabel, kuhu andmed sisestatakse
create table esinejad(
id int primary key identity(1,1),
esineja varchar(30),
koht varchar(50),
kuupaev date)

--tabel, kuhu sisestatakse päästikute andmed
create table ajakiri(
id int primary key identity(1,1),
toiming varchar(30),
andmed varchar(100),
kasutaja varchar(30),
kuupaev datetime)

--Päästik, mis näitab, kes ja millal andmeid lisas
create trigger esinejaLisamine
on esinejad
for insert
as
insert into ajakiri(toiming, andmed, kasutaja, kuupaev)
select
'kasutaja lisas andmeid',
concat('Esineja: ', inserted.esineja, ', Koht: ', inserted.koht, ', Kuupäev: ', inserted.kuupaev),
system_user,
getdate()
from inserted


insert into esinejad
values('Eminem', 'Tallinn Song Festival Grounds', '2026-07-12')

insert into esinejad
values('Drake', 'Tallinn Song Festival Grounds', '2026-08-02')

insert into esinejad
values('MORGENSTERN', 'Helitehas', '2026-09-17')

select * from esinejad
select * from ajakiri

--Päästik, mis näitab, kes ja millal andmed kustutas.
create trigger esinejaKustutamine
on esinejad --tabelinimi, mis on vaja jälgida
for delete
as
insert into ajakiri(toiming, andmed, kasutaja, kuupaev) --kuhu
select
'kasutaja kustutas andmeid', --tegevus
concat('Esineja: ', deleted.esineja, ', Koht: ', deleted.koht, ', Kuupäev: ', deleted.kuupaev), --andmed
system_user, --kasutaja
getdate() --aeg
from deleted

drop trigger esinejaKustutamine

insert into esinejad
values('Ivan Zolo', 'Kristiine Keskus', '2026-06-10')

delete from esinejad where esineja='Ivan Zolo'

select * from esinejad
select * from ajakiri

--Päästik, mis näitab andmete muutust
create trigger esinejaUuendamine
on esinejad
for update
as
begin
insert into ajakiri(toiming, andmed, kasutaja, kuupaev)
select
'kasutaja uuendas andmeid',
concat('Vanad andmed - Esineja: ', deleted.esineja, ', Koht: ', deleted.koht, ', Kuupäev: ', deleted.kuupaev,
', Uued andmed - Esineja: ', inserted.esineja, ', Koht: ', inserted.koht, ', Kuupäev: ', inserted.kuupaev),
system_user,
getdate()
from deleted inner join inserted
on deleted.id=inserted.id;

update esinejad set kuupaev='2026-07-20' where id=1

select * from esinejad
select * from ajakiri

--Loome ühe ühise päästiku, mis näitab, kas andmeid on lisatud või kustutatud.
disable trigger esinejaLisamine on esinejad
disable trigger esinejaKustutamine on esinejad

create trigger esinejadInsDel
ON esinejad
FOR INSERT, DELETE
AS
BEGIN
SET NOCOUNT ON;
	insert into ajakiri(toiming, andmed, kasutaja, kuupaev)
	select
	'kasutaja lisas andmeid',
	concat('Esineja: ', inserted.esineja, ', Koht: ', inserted.koht, ', Kuupäev: ', inserted.kuupaev),
	system_user,
	getdate()
	from inserted

	union all

	select
	'kasutaja kustutas andmeid',
	concat('Esineja: ', deleted.esineja, ', Koht: ', deleted.koht, ', Kuupäev: ', deleted.kuupaev),
	system_user,
	getdate()
	from deleted
end;

insert into esinejad
values('Nikita', 'TTHK', '2026-07-03')

delete from esinejad where esineja='Nikita'

select * from esinejad
select * from ajakiri

--Kuva päästikud
EXEC sp_helptext esinejaLisamine
EXEC sp_helptrigger 'esinejad', 'insert'
EXEC sp_helptext esinejadInsDel

grant select, insert, delete, update on esinejad to sekretarRuslan2
deny select, insert, delete, update on ajakiri to sekretarRuslan2
```
