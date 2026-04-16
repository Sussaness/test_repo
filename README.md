CREATE DATABASE EmploymentService;
GO

USE EmploymentService;
GO

-- Таблица безработных
CREATE TABLE Unemployed (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    FullName NVARCHAR(200) NOT NULL,
    RegistrationAddress NVARCHAR(500) NOT NULL,
    ActualAddress NVARCHAR(500) NULL,
    Phones NVARCHAR(200) NULL,
    MaritalStatus NVARCHAR(100) NULL,
    FamilyComposition NVARCHAR(MAX) NULL,
    Education NVARCHAR(100) NULL,
    WorkExperienceYears FLOAT NULL,
    LastJobPlace NVARCHAR(200) NULL,
    LastJobPosition NVARCHAR(200) NULL,
    LastSalary FLOAT NULL,
    DesiredJobType NVARCHAR(200) NULL,
    DesiredMinSalary FLOAT NULL,
    DesiredEmploymentType NVARCHAR(10) CHECK (DesiredEmploymentType IN ('Full', 'Part')),
    RegistrationDate DATE NOT NULL,
    HasPreviousJob BIT NOT NULL,
    BenefitAmount FLOAT NULL
);

-- Таблица вакансий
CREATE TABLE Vacancy (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    CompanyName NVARCHAR(200) NOT NULL,
    Position NVARCHAR(200) NOT NULL,
    EmploymentType NVARCHAR(10) CHECK (EmploymentType IN ('Full', 'Part')),
    RequiredEducation NVARCHAR(100) NULL,
    RequiredMinAge INT NULL,
    RequiredMaxAge INT NULL,
    RequiredWorkExpYears FLOAT NULL,
    RequiredProfileExpYears FLOAT NULL,
    Salary FLOAT NOT NULL,
    HasBonuses BIT NULL,
    CreatedDate DATE NOT NULL
);

-- Добавим тестовые вакансии
INSERT INTO Vacancy (CompanyName, Position, EmploymentType, RequiredEducation, RequiredMinAge, RequiredMaxAge, RequiredWorkExpYears, RequiredProfileExpYears, Salary, HasBonuses, CreatedDate)
VALUES 
(N'ООО Рога и Копыта', N'Программист C#', 'Full', N'высшее', 22, 45, 1.0, 1.0, 80000, 1, CAST(GETDATE() AS DATE)),
(N'Такси Везунчик', N'Водитель', 'Part', N'среднее', 21, 60, 0.5, 0.0, 40000, 0, CAST(GETDATE() AS DATE));
