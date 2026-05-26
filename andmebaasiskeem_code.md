```sql
create database RetseptRus
use RetseptRus

--tabel kasutaja
create table kasutaja(
kasutaja_id int primary key identity(1,1),
eesnimi varchar(50),
perenimi varchar(50),
email varchar(50))

insert into kasutaja
values('Devid', 'Babjak', 'devid.babjak@gmail.com')

select * from kasutaja

--tabel kategooria
create table kategooria(
kategooria_id int primary key identity(1,1),
kategooria_nimi varchar(50))

insert into kategooria
values('Liha')

select * from kategooria

--tabel toiduaine
create table toiduaine(
toiduaine_id int primary key identity(1,1),
toiduaine_nimi varchar(100))

insert into toiduaine
values('Kartul')

select * from toiduaine

--tabel yhik
create table yhik(
yhik_id int primary key identity(1,1),
yhik_nimi varchar(100))

insert into yhik
values('sp')

select * from yhik

--tabel retsept
create table retsept(
retsept_id int primary key identity(1,1),
retsept_nimi varchar(100),
kirjeldus varchar(200),
juhend varchar(500),
sisestatud_kp date,
kasutaja_id int,
foreign key (kasutaja_id) references kasutaja(kasutaja_id),
kategooria_id int,
foreign key (kategooria_id) references kategooria(kategooria_id))

insert into retsept
values('Pepperoni Pitsa', 'Maitsev pitsa', 'Küpseta 20 minutit', '2026-06-1', 1, 1)
insert into retsept
values('Kanapuljong', 'Soe supp', 'Keeda 40 minutit', '2025-06-02', 2, 2)
insert into retsept
values('Praetud kala', 'Krõbe kala', 'Prae pannil', '2025-06-03', 3, 3)
insert into retsept
values('Šokolaadikook', 'Magus kook', 'Küpseta ahjus', '2026-06-04', 4, 4)
insert into retsept
values('Steik', 'Veiseliha steik', 'Prae medium', '2026-06-05', 5, 5)

select * from retsept

--tabel koostis
create table koostis(
koostis_id int primary key identity(1,1),
kogus int,
retsept_id int,
foreign key (retsept_id) references retsept(retsept_id),
toiduaine_id int,
foreign key (toiduaine_id) references toiduaine(toiduaine_id),
yhik_id int,
foreign key (yhik_id) references yhik(yhik_id))

insert into koostis
values(500, 1, 1, 1);
insert into koostis
values(2, 1, 2, 4);
insert into koostis
values(300, 1, 3, 1);
insert into koostis
values(1, 2, 4, 2);
insert into koostis
values(5, 2, 5, 4);

select * from koostis

--tabel tehtud
create table tehtud(
tehtud_id int primary key identity(1,1),
tehtud_kp date,
retsept_id int,
foreign key (retsept_id) references retsept(retsept_id))

insert into tehtud
values('2026-07-10', 1)
insert into tehtud
values('2026-07-12', 2)
insert into tehtud
values('2026-07-14', 3)
insert into tehtud
values('2026-07-17', 4)
insert into tehtud
values('2026-07-20', 5)

select * from tehtud

--Protseduurid
create Procedure lisatoiduaine
@uustoiduaine varchar(50)
AS
BEGIN
	insert into toiduaine(toiduaine_nimi)
	values(@uustoiduaine)
	select * from toiduaine
END;
exec lisatoiduaine ketchup

create Procedure lisakasutaja
@uuseesnimi varchar(50),
@uusperenimi varchar(50),
@uusemail varchar(150)
AS
BEGIN
	insert into kasutaja(eesnimi, perenimi, email)
	values(@uuseesnimi, @uusperenimi, @uusemail)
	select * from kasutaja
END;
exec lisakasutaja 'Arseni', 'Rumjantsev', 'ars.rumjantsev@gmail.com' 

create procedure muudatabel
@tegevus varchar(50),
@tabelinimi varchar(50),
@veerunimi varchar(50),
@tyyp varchar(50)=null
AS
BEGIN
    DECLARE @sqltegevus varchar(max);

    SET @sqltegevus = CASE 
        WHEN @tegevus = 'add' THEN 
            CONCAT('ALTER TABLE ', @tabelinimi, ' ADD ', @veerunimi, ' ', @tyyp)
        WHEN @tegevus = 'drop' THEN 
            CONCAT('ALTER TABLE ', @tabelinimi, ' DROP COLUMN ', @veerunimi)
		WHEN @tegevus = 'change' THEN
			CONCAT('ALTER TABLE ', @tabelinimi, ' ALTER COLUMN ', @veerunimi, ' ', @tyyp)
	END;
    PRINT @sqltegevus;
    EXEC (@sqltegevus);
END;

EXEC muudatabel 'drop', 'toiduaine', 'toiduaine_kogus'
select * from 

--select-päringud
SELECT kasutaja.eesnimi, retsept.retsept_nimi
FROM kasutaja, retsept
WHERE kasutaja.kasutaja_id = retsept.kasutaja_id;

--oma tabel
create table kasutaja_tehtud(
kasutaja_id int,
foreign key (kasutaja_id) references kasutaja(kasutaja_id),
retsept_id int,
foreign key (retsept_id) references retsept(retsept_id),
kogus int)

insert into kasutaja_tehtud
values(1, 1, 64)
insert into kasutaja_tehtud
values(2, 2, 178)
insert into kasutaja_tehtud
values(3, 3, 23)
insert into kasutaja_tehtud
values(4, 4, 93)
insert into kasutaja_tehtud
values(1, 5, 13)

select * from kasutaja_tehtud

--oma protseduur
create procedure lisakasutajatehtud
@uuskasutaja_id int,
@uusretsept_id int,
@uuskogus int
AS
BEGIN
	insert into kasutaja_tehtud(kasutaja_id, retsept_id, kogus)
	values(@uuskasutaja_id, @uusretsept_id, @uuskogus)
	select * from kasutaja_tehtud
END;

exec lisakasutajatehtud 2, 2, 128

create procedure deletekasutajatehtud
@id int
AS
BEGIN
	select * from kasutaja_tehtud;
	delete from kasutaja_tehtud where kasutaja_id=@id
	select * from kasutaja_tehtud;
END;
exec deletekasutajatehtud 2
--staffR grants
GRANT SELECT ON toiduaine TO staffR;
GRANT SELECT ON kategooria TO staffR;
GRANT SELECT ON kasutaja TO staffR;

GRANT INSERT ON toiduaine TO staffR;
GRANT INSERT ON kategooria TO staffR;

DENY DELETE ON toiduaine TO staffR;
DENY DELETE ON kategooria TO staffR;

DENY UPDATE ON toiduaine TO staffR;
DENY UPDATE ON kategooria TO staffR;

--managerR grants
GRANT SELECT ON kasutaja TO managerR;
GRANT SELECT ON kategooria TO managerR;
GRANT SELECT ON toiduaine TO managerR;

GRANT SELECT ON retsept TO managerR;
GRANT UPDATE ON retsept TO managerR;
GRANT DELETE ON retsept TO managerR;

GRANT SELECT ON koostis TO managerR;
GRANT UPDATE ON koostis TO managerR;
GRANT DELETE ON koostis TO managerR;

GRANT SELECT ON yhik TO managerR;
GRANT SELECT ON tehtud TO managerR;
GRANT SELECT ON kasutaja_tehtud TO managerR;

DENY INSERT ON toiduaine TO managerR;
DENY INSERT ON kasutaja TO managerR;
```
