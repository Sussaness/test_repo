using System;
using System.Collections.Generic;
using System.Data.SqlClient;
using System.Xml.Linq;

namespace EmploymentService
{
    class Program
    {
        static string connStr = "Server=localhost;Database=EmploymentService;Integrated Security=True;TrustServerCertificate=True;";

        static void Main()
        {
            Console.WriteLine("1 - постановка на учёт, 2 - выход");
            if (Console.ReadLine() == "1") Register();
        }

        static void Register()
        {
            Console.Write("ФИО: "); string fio = Console.ReadLine();
            Console.Write("Адрес рег: "); string regAddr = Console.ReadLine();
            Console.Write("Образование: "); string edu = Console.ReadLine();
            Console.Write("Стаж (лет): "); double exp = double.Parse(Console.ReadLine());
            Console.Write("Была ли работа? (да/нет): "); bool hasJob = Console.ReadLine().ToLower() == "да";
            double lastSalary = 0;
            if (hasJob) { Console.Write("Последняя зарплата: "); lastSalary = double.Parse(Console.ReadLine()); }
            Console.Write("Желаемая мин зарплата: "); double minSal = double.Parse(Console.ReadLine());
            Console.Write("Тип занятости (Full/Part): "); string empType = Console.ReadLine();

            double benefit = hasJob ? lastSalary * 0.8 : 19242;

            using var conn = new SqlConnection(connStr);
            conn.Open();
            string sql = @"INSERT INTO Unemployed (FullName, RegistrationAddress, Education, WorkExperienceYears, HasPreviousJob, LastSalary, DesiredMinSalary, DesiredEmploymentType, RegistrationDate, BenefitAmount)
                           VALUES (@fn, @ra, @edu, @exp, @hpj, @ls, @dms, @det, GETDATE(), @ben);
                           SELECT SCOPE_IDENTITY();";
            using var cmd = new SqlCommand(sql, conn);
            cmd.Parameters.AddWithValue("@fn", fio);
            cmd.Parameters.AddWithValue("@ra", regAddr);
            cmd.Parameters.AddWithValue("@edu", edu);
            cmd.Parameters.AddWithValue("@exp", exp);
            cmd.Parameters.AddWithValue("@hpj", hasJob ? 1 : 0);
            cmd.Parameters.AddWithValue("@ls", lastSalary);
            cmd.Parameters.AddWithValue("@dms", minSal);
            cmd.Parameters.AddWithValue("@det", empType);
            cmd.Parameters.AddWithValue("@ben", benefit);
            int id = Convert.ToInt32(cmd.ExecuteScalar());

            Console.WriteLine($"Безработный {fio} поставлен на учёт, пособие {benefit}");

            // Поиск вакансий
            string findVac = @"SELECT CompanyName, Position, Salary FROM Vacancy 
                                WHERE (RequiredEducation IS NULL OR RequiredEducation = @edu)
                                  AND Salary >= @minSal
                                  AND (RequiredWorkExpYears IS NULL OR RequiredWorkExpYears <= @exp)";
            using var cmd2 = new SqlCommand(findVac, conn);
            cmd2.Parameters.AddWithValue("@edu", edu);
            cmd2.Parameters.AddWithValue("@minSal", minSal);
            cmd2.Parameters.AddWithValue("@exp", exp);
            var vacancies = new List<string>();
            using var r = cmd2.ExecuteReader();
            while (r.Read())
                vacancies.Add($"{r[0]} - {r[1]} - {r[2]} руб.");

            if (vacancies.Count == 0) Console.WriteLine("Нет подходящих вакансий");
            else Console.WriteLine("Вакансии:\n" + string.Join("\n", vacancies));

            Console.Write("Экспорт в XML? (да/нет): ");
            if (Console.ReadLine().ToLower() == "да")
            {
                var doc = new XDocument(
                    new XElement("Vacancies",
                        new XAttribute("For", fio),
                        vacancies.Select(v => new XElement("Vacancy", v))
                    ));
                doc.Save("vacancies.xml");
                Console.WriteLine("Сохранён vacancies.xml");
            }
            Console.ReadKey();
        }
    }
}
