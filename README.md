#include <stdio.h>
#include <math.h>
#include <stdlib.h>
#include <locale.h>

#define MAX_ITERATIONS 1000
#define EPS 1e-10

// Функция для вычисления f(x) с проверкой области определения
double f(double x)
{
    if (x < -1)
    {
        // arcsin(x/2) определен при |x/2| <= 1, т.е. |x| <= 2
        if (x < -2 || x > 2)
        {
            printf("Ошибка: x = %f вне области определения arcsin(x/2)\n", x);
            return NAN;
        }
        return asin(x / 2);
    }
    else if (x < 0)
    {
        // (e^x - x - 1)/x^2 определен при x != 0
        if (fabs(x) < EPS)
        {
            printf("Ошибка: x = %f слишком близко к 0 для второй формулы\n", x);
            return NAN;
        }
        return (exp(x) - x - 1) / (x * x);
    }
    else
    {
        // Ряд определен для всех x >= 0
        double sum = 0.0;
        for (int n = 0; n <= 11; n++)
        {
            double numerator = pow(-1, n) * pow(x, 2 * n);
            double denominator = (2 * n + 1) * log(n + 2);
            sum += numerator / denominator;
        }
        return sum;
    }
}

// Операция 1: Значение функции в точке
void value_at_point()
{
    double x;
    printf("Введите точку x: ");
    if (scanf("%lf", &x) != 1)
    {
        printf("Ошибка: некорректный ввод\n");
        while (getchar() != '\n') ; // Очистка буфера
        return;
    }

    double result = f(x);
    if (!isnan(result))
    {
        printf("f(%f) = %f\n", x, result);
    }
}

// Операция 2: Таблица значений на интервале
void table_of_values()
{
    double a, b;
    int n;

    printf("Введите начало интервала a: ");
    if (scanf("%lf", &a) != 1)
    {
        printf("Ошибка: некорректный ввод\n");
        while (getchar() != '\n') ;
        return;
    }

    printf("Введите конец интервала b: ");
    if (scanf("%lf", &b) != 1)
    {
        printf("Ошибка: некорректный ввод\n");
        while (getchar() != '\n') ;
        return;
    }

    if (a >= b)
    {
        printf("Ошибка: a должно быть меньше b\n");
        return;
    }

    printf("Введите количество точек n: ");
    if (scanf("%d", &n) != 1 || n <= 0)
    {
        printf("Ошибка: некорректное количество точек\n");
        while (getchar() != '\n') ;
        return;
    }

    printf("\nТаблица значений:\n");
    printf("x\t\tf(x)\n");
    printf("-----------------------\n");

    double step = (b - a) / (n - 1);
    for (int i = 0; i < n; i++)
    {
        double x = a + i * step;
        double result = f(x);
        if (!isnan(result))
        {
            printf("%.6f\t%.6f\n", x, result);
        }
        else
        {
            printf("%.6f\tне определено\n", x);
        }
    }
}

// Операция 3: Вычисление определенного интеграла методом прямоугольников
void definite_integral()
{
    double a, b;
    int n;

    printf("Введите нижний предел интегрирования a: ");
    if (scanf("%lf", &a) != 1)
    {
        printf("Ошибка: некорректный ввод\n");
        while (getchar() != '\n') ;
        return;
    }

    printf("Введите верхний предел интегрирования b: ");
    if (scanf("%lf", &b) != 1)
    {
        printf("Ошибка: некорректный ввод\n");
        while (getchar() != '\n') ;
        return;
    }

    if (a >= b)
    {
        printf("Ошибка: a должно быть меньше b\n");
        return;
    }

    printf("Введите количество разбиений n: ");
    if (scanf("%d", &n) != 1 || n <= 0)
    {
        printf("Ошибка: некорректное количество разбиений\n");
        while (getchar() != '\n') ;
        return;
    }

    double h = (b - a) / n;
    double sum = 0.0;
    int valid_points = 0;

    // Метод средних прямоугольников
    for (int i = 0; i < n; i++)
    {
        double x_mid = a + (i + 0.5) * h;
        double fx = f(x_mid);
        if (!isnan(fx))
        {
            sum += fx;
            valid_points++;
        }
    }

    if (valid_points == 0)
    {
        printf("Ошибка: функция не определена на всем интервале\n");
        return;
    }

    double integral = sum * h;
    printf("Определенный интеграл от %f до %f = %f\n", a, b, integral);
    printf("(использовано %d из %d точек)\n", valid_points, n);
}

// Операция 4: Поиск корней функции на интервале
void find_roots()
{
    double a, b;

    printf("Введите начало интервала a: ");
    if (scanf("%lf", &a) != 1)
    {
        printf("Ошибка: некорректный ввод\n");
        while (getchar() != '\n') ;
        return;
    }

    printf("Введите конец интервала b: ");
    if (scanf("%lf", &b) != 1)
    {
        printf("Ошибка: некорректный ввод\n");
        while (getchar() != '\n') ;
        return;
    }

    if (a >= b)
    {
        printf("Ошибка: a должно быть меньше b\n");
        return;
    }

    int n = 1000; // Количество проверяемых точек
    double step = (b - a) / n;
    int root_count = 0;

    printf("\nПоиск корней на интервале [%f, %f]:\n", a, b);

    // Простой поиск смены знака
    double prev_x = a;
    double prev_fx = f(a);

    for (int i = 1; i <= n; i++)
    {
        double x = a + i * step;
        double fx = f(x);

        if (isnan(fx)) continue;
        if (isnan(prev_fx))
        {
            prev_x = x;
            prev_fx = fx;
            continue;
        }

        // Если функция меняет знак
        if (prev_fx * fx <= 0)
        {
            // Уточнение корня методом деления пополам
            double left = prev_x;
            double right = x;
            double root = (left + right) / 2;
            int iterations = 0;

            while (fabs(right - left) > EPS && iterations < MAX_ITERATIONS)
            {
                root = (left + right) / 2;
                double f_root = f(root);

                if (isnan(f_root)) break;

                if (f(left) * f_root <= 0)
                {
                    right = root;
                }
                else
                {
                    left = root;
                }
                iterations++;
            }

            if (!isnan(f(root)) && fabs(f(root)) < 0.01)
            { // Допуск для корня
                printf("Корень: x = %f, f(x) = %f\n", root, f(root));
                root_count++;
            }
        }

        prev_x = x;
        prev_fx = fx;
    }

    if (root_count == 0)
    {
        printf("Корни не найдены на заданном интервале\n");
    }
}

// Операция 5: Производная функции в точке
void derivative_at_point()
{
    double x;

    printf("Введите точку x: ");
    if (scanf("%lf", &x) != 1)
    {
        printf("Ошибка: некорректный ввод\n");
        while (getchar() != '\n') ;
        return;
    }

    // Численное дифференцирование
    double h = 1e-8;
    double f_plus, f_minus;

    f_plus = f(x + h);
    f_minus = f(x - h);

    if (isnan(f_plus) || isnan(f_minus))
    {
        printf("Ошибка: функция не определена в окрестности точки %f\n", x);
        return;
    }

    double derivative = (f_plus - f_minus) / (2 * h);
    printf("f'(%f) ≈ %f\n", x, derivative);
}

// Главное меню
void print_menu()
{
    printf("\n=== Меню операций ===\n");
    printf("1. Значение функции в точке\n");
    printf("2. Таблица значений на интервале\n");
    printf("3. Вычисление определенного интеграла\n");
    printf("4. Поиск корней функции на интервале\n");
    printf("5. Производная функции в точке\n");
    printf("0. Выход\n");
    printf("Выберите операцию: ");
}

int main()
{
    setlocale(LC_CTYPE, "RUS");
    int choice;

    printf("Программа для работы с кусочно-заданной функцией\n");

    do
    {
        print_menu();
        if (scanf("%d", &choice) != 1)
        {
            printf("Ошибка: некорректный ввод\n");
            while (getchar() != '\n') ;
            continue;
        }

        switch (choice)
        {
            case 1:
                value_at_point();
                break;
            case 2:
                table_of_values();
                break;
            case 3:
                definite_integral();
                break;
            case 4:
                find_roots();
                break;
            case 5:
                derivative_at_point();
                break;
            case 0:
                printf("Выход из программы.\n");
                break;
            default:
                printf("Неверный выбор. Попробуйте снова.\n");
                break;
        }
    } while (choice != 0);

    return 0;
}
