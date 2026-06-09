create database reisimineR
use reisimineR
--1. kasutaja reisijaRuslan loodud
--2,3. tabelite loomine

create table lennujaam(
lennujaam_id int primary key identity(1,1),
lennujaama_nimi varchar(50),
linn varchar(30))

insert into lennujaam
values('Baltic', 'Tallinn'), ('Transert', 'Helsinki'), ('Nemi', 'Haapsalu')

select * from lennujaam

create table lend(
lend_id int primary key identity(1,1),
lennu_number int,
valjumisaeg time default '12:00:00',
lennujaam_id int,
foreign key(lennujaam_id) references lennujaam(lennujaam_id))


insert into lend
values(12834, '16:30:00', 1), (38193, '14:00:00', 2), (12846, '12:00:00', 3)

select * from lend

create table reisija(
reisija_id int primary key identity(1,1),
nimi varchar(30),
pitetinumber varchar(20),
lend_id int,
foreign key(lend_id) references lend(lend_id))

insert into reisija
values('Ruslan', 129465, 1), ('Maksim', 837584, 2), ('Arsen', 465720, 3)

select * from reisija

--kasutaja õigused
grant create table to reisijaRuslan 

deny alter on object::reisimineR to reisijaRuslan

grant alter to reisijaRuslan

grant select to reisijaRuslan
grant insert, delete on reisija to reisijaRuslan
grant insert, delete on lend to reisijaRuslan

--6. tabel logi
create table logi(
id int primary key identity(1,1),
kasutaja varchar(30),
kuupaev datetime,
sisestatud_andmed varchar(200))

select * from logi

--kustutamine triger
CREATE TRIGGER lendKustutamine
ON lend --tabelinimi, mis on vaja jälgida
FOR DELETE
AS
INSERT INTO logi(kasutaja, kuupaev, sisestatud_andmed)
SELECT
SYSTEM_USER, --kasutaja mis on sisselogitud serverisse
GETDATE(),  --aeg
concat ('Deleted: lennu number:', deleted.lennu_number, ', väljumisaeg:', deleted.valjumisaeg, ', lennujaama id:', deleted.lennujaam_id)  --andmed tabelisy linnad
FROM deleted;

insert into lend
values(12834, '16:30:00', 1)

select * from lend

delete from lend where lend_id=7

select * from logi

--lisamine triger
CREATE TRIGGER lendLisamine
ON lend --tabelinimi, mis on vaja jälgida
FOR INSERT
AS
INSERT INTO logi(kasutaja, kuupaev, sisestatud_andmed)
SELECT
SYSTEM_USER, --kasutaja mis on sisselogitud serverisse
GETDATE(),  --aeg
concat ('Inserted: lennu number:', inserted.lennu_number, ', väljumisaeg:', inserted.valjumisaeg, ', lennujaama id:', inserted.lennujaam_id)  --andmed tabelisy linnad
FROM inserted;

insert into lend
values(12834, '16:30:00', 1)

select * from logi

drop trigger lendLisamine

--11. Protseduurid
--esineme protseduur(andmete lisamine tabelisse reisija)
create procedure LisaReisija
@uus_nimi varchar(30),
@uus_pitetinumber varchar(20),
@uus_lend_id int
as
begin
	select * from reisija
	insert into reisija
	values(@uus_nimi, @uus_pitetinumber, @uus_lend_id)
	select * from reisija
end

exec LisaReisija 'testreisija', 00002, 2

--teine protseduur(andmete kustutamine tabelist lennujaam)
CREATE procedure KustutaLennujaam
@kustuta_id int
AS
BEGIN
	SELECT * FROM lennujaam;
	DELETE FROM lennujaam WHERE lennujaam_id=@kustuta_id
	SELECT * FROM lennujaam;
END;

--kutse
insert into lennujaam
values('mememe', 'nenene')

EXEC KustutaLennujaam 5

--kolmas protseduur(muuta aega id-ga)
create procedure UusAeg
@id int,
@uus_aeg time
as
begin
	select * from lend;
	update lend set valjumisaeg=@uus_aeg where lend_id=@id;
	select * from lend;
end

exec UusAeg 10, '21:30:00'

--12. select päringud
--esimene päring(näitab, kes konkreetse lennuga läheb)


drop view kes_on_reisil

CREATE VIEW kes_on_reisil 
AS
SELECT 
    l.lennu_number,
    r.nimi AS reisija_nimi
FROM lend l
INNER JOIN reisija r ON l.lend_id = r.lend_id;

select * from kes_on_reisil

--teine päring(näitab inimest ja tema lennu väljumisaeg)
-- Tingimusvaade: näitab reisija nime ja tema reisi väljumisaega
CREATE VIEW ReisiAeg 
AS
SELECT 
    r.nimi AS reisija_nimi,
    l.valjumisaeg
FROM reisija r
INNER JOIN lend l ON r.lend_id = l.lend_id;

select * from ReisiAeg

--kolmas päring(näitab reisija nime ja tema reisi numbri
-- Tingimusvaade: näitab reisija nime ja lennujaama nime
CREATE VIEW reisija_lennujaam 
AS
SELECT 
    r.nimi AS reisija_nimi,
    lenn.lennujaama_nimi
FROM reisija r
INNER JOIN lend l ON r.lend_id = l.lend_id
INNER JOIN lennujaam lenn ON l.lennujaam_id = lenn.lennujaam_id;

select * from reisija_lennujaam

--13. oma tegevus
--See on protseduur, mis võimaldab teil valida tabeli, valida toimingu ("INSERT, DELETE, UPDATE") ja käivitada need vastavalt määratud parameetritele.
CREATE PROCEDURE omategevus
@tegevus varchar(10),
@tabelinimi varchar(30),
@tahendus1 varchar(30) = NULL,
@tahendus2 varchar(30) = NULL,
@tahendus4 int = NULL,
@delete_id varchar(30) = NULL,
@id int = NULL,
@uus_tahendus varchar(30) = NULL,
@veerg1 varchar(30) = NULL,
@veerg2 varchar(30) = NULL,
@vana_tahendus varchar(30) = NULL
AS
BEGIN
    DECLARE @sqltegevus varchar(max);
	SET @sqltegevus = CASE
	when @tegevus = 'INSERT' then
		CONCAT('INSERT INTO ', @tabelinimi, ' VALUES (''', @tahendus1, ''',''', @tahendus2, ''',', @tahendus4, ')')
	when @tegevus = 'DELETE' then 
		concat('DELETE FROM ', @tabelinimi, ' WHERE ', @delete_id, '=', @id)
	when @tegevus = 'UPDATE' then
		CONCAT('UPDATE ', @tabelinimi, ' SET ', @veerg1, ' = ''', @uus_tahendus, ''' WHERE ', @veerg2, ' = ''', @vana_tahendus, '''')
	END;

    PRINT @sqltegevus;
    EXEC (@sqltegevus);
END;

drop procedure omategevus

--insert
EXEC omategevus
    @tegevus='INSERT',
    @tabelinimi='procedure_test',
    @tahendus1='mememe',
    @tahendus2='nenene',
    @tahendus4=50;

	--delete
EXEC omategevus
    @tegevus='DELETE',
    @tabelinimi='procedure_test',
    @delete_id='test_id',
    @id=3;

	--update
EXEC omategevus
    @tegevus='UPDATE',
    @tabelinimi='procedure_test',
    @veerg1='test_nimi',
    @uus_tahendus='kekeke',
    @veerg2='test_nimi',
    @vana_tahendus='test_nimi';

select * from procedure_test


create table procedure_test(
test_id int primary key identity(1,1),
test_nimi varchar(30),
test_kirjeldus varchar(30),
test_number int null)

insert into procedure_test
values('test nimi', 'test kirjeldus', 15)

select * from procedure_test

select * from lennujaam
select * from lend
select * from reisija

drop table test


create table lend3(
lend3_id int primary key identity(1,1),
lennu3_number int,
valjumisaeg3 time(0),
lennujaam_id int,
foreign key(lennujaam_id) references lennujaam(lennujaam_id))

insert into lend3
values(12834, '16:30:00', 1)

select * from lend3
