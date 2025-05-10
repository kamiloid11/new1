1. Реализация алгоритма Рабина-Карпа на Java

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class RabinKarp {
    private static final int PRIME = 101; // Простое число для хеширования
    private static long iterations = 0; // Счетчик итераций

    public static List<Integer> search(String text, String pattern) {
        iterations = 0; // Сброс счетчика итераций
        List<Integer> occurrences = new ArrayList<>();
        int n = text.length();
        int m = pattern.length();
        
        if (m == 0 || n < m) return occurrences;

        // Вычисляем хеш образца и первого окна текста
        long patternHash = createHash(pattern, m);
        long textHash = createHash(text, m);
        
        // Проходим по тексту
        for (int i = 0; i <= n - m; i++) {
            iterations++; // Увеличиваем счетчик итераций
            
            // Проверяем хеши, если совпадают - проверяем символы
            if (patternHash == textHash && checkEqual(text, i, i + m - 1, pattern)) {
                occurrences.add(i);
            }
            
            // Вычисляем хеш для следующего окна текста
            if (i < n - m) {
                textHash = recalculateHash(text, i, i + m, textHash, m);
            }
        }
        
        return occurrences;
    }
    
    private static long createHash(String str, int length) {
        long hash = 0;
        for (int i = 0; i < length; i++) {
            hash += str.charAt(i) * Math.pow(PRIME, i);
            iterations++; // Учитываем итерации при вычислении хеша
        }
        return hash;
    }
    
    private static long recalculateHash(String str, int oldIndex, int newIndex, long oldHash, int patternLength) {
        long newHash = oldHash - str.charAt(oldIndex);
        newHash = newHash / PRIME;
        newHash += str.charAt(newIndex) * Math.pow(PRIME, patternLength - 1);
        iterations++; // Учитываем итерации при пересчете хеша
        return newHash;
    }
    
    private static boolean checkEqual(String text, int start, int end, String pattern) {
        for (int i = start; i <= end; i++) {
            iterations++; // Учитываем итерации при проверке символов
            if (text.charAt(i) != pattern.charAt(i - start)) {
                return false;
            }
        }
        return true;
    }
    
    public static long getIterations() {
        return iterations;
    }
    
    // Генератор случайных строк
    public static String generateRandomString(int length) {
        String characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        Random random = new Random();
        StringBuilder sb = new StringBuilder(length);
        
        for (int i = 0; i < length; i++) {
            sb.append(characters.charAt(random.nextInt(characters.length())));
        }
        
        return sb.toString();
    }
}

2. Генерация наборов входных данных

import java.io.FileWriter;
import java.io.IOException;
import java.util.Random;

public class DataGenerator {
    public static void main(String[] args) {
        Random random = new Random();
        int numDatasets = 100;
        int minSize = 100;
        int maxSize = 10000;
        
        try (FileWriter writer = new FileWriter("test_data.txt")) {
            for (int i = 0; i < numDatasets; i++) {
                int textSize = minSize + random.nextInt(maxSize - minSize + 1);
                int patternSize = 1 + random.nextInt(textSize / 10); // Паттерн ~10% от текста
                
                String text = RabinKarp.generateRandomString(textSize);
                String pattern = RabinKarp.generateRandomString(patternSize);
                
                writer.write(textSize + "|" + patternSize + "|" + text + "|" + pattern + "\n");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

3. Тестирование алгоритма и измерение времени

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.List;

public class RabinKarpTester {
    public static void main(String[] args) {
        try (BufferedReader reader = new BufferedReader(new FileReader("test_data.txt"))) {
            System.out.println("Size\tPatternSize\tTime(ns)\tIterations");
            
            String line;
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split("\\|", 4);
                int textSize = Integer.parseInt(parts[0]);
                int patternSize = Integer.parseInt(parts[1]);
                String text = parts[2];
                String pattern = parts[3];
                
                // Измеряем время выполнения
                long startTime = System.nanoTime();
                List<Integer> result = RabinKarp.search(text, pattern);
                long endTime = System.nanoTime();
                long duration = endTime - startTime;
                
                // Получаем количество итераций
                long iterations = RabinKarp.getIterations();
                
                System.out.println(textSize + "\t" + patternSize + "\t" + duration + "\t" + iterations);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

4. Анализ результатов и построение графиков
Для построения графиков можно использовать Python с библиотеками matplotlib и pandas:

import pandas as pd
import matplotlib.pyplot as plt

# Загрузка данных
data = pd.read_csv('results.csv', sep='\t')  # Файл с результатами тестирования

# График времени выполнения от размера текста
plt.figure(figsize=(12, 6))
plt.scatter(data['Size'], data['Time(ns)'], alpha=0.5, label='Фактическое время')
plt.xlabel('Размер текста')
plt.ylabel('Время (наносекунды)')
plt.title('Зависимость времени выполнения от размера текста')
plt.grid(True)

# Линия идеальной линейной сложности O(n)
max_time = data['Time(ns)'].max()
ideal_n = [max_time * (n / data['Size'].max()) for n in data['Size']]
plt.plot(data['Size'], ideal_n, 'r--', label='Идеальная O(n)')
plt.legend()
plt.show()

# График итераций от размера текста
plt.figure(figsize=(12, 6))
plt.scatter(data['Size'], data['Iterations'], alpha=0.5, label='Фактические итерации')
plt.xlabel('Размер текста')
plt.ylabel('Количество итераций')
plt.title('Зависимость количества итераций от размера текста')
plt.grid(True)

# Линия идеальной линейной сложности O(n)
max_iter = data['Iterations'].max()
ideal_iter = [max_iter * (n / data['Size'].max()) for n in data['Size']]
plt.plot(data['Size'], ideal_iter, 'r--', label='Идеальная O(n)')
plt.legend()
plt.show()

5. Оценка сложности алгоритма и анализ результатов

Теоретическая оценка сложности:

Алгоритм Рабина-Карпа имеет:
Средняя и лучшая временная сложность: O(n + m), где n - длина текста, m - длина образца
Худшая временная сложность: O(n*m) (когда все подстроки имеют одинаковый хеш)
Пространственная сложность: O(1) (не считая хранения входных данных)

Анализ результатов:

1. Временные характеристики:

На графике времени выполнения видно, что зависимость близка к линейной (O(n)), что соответствует теоретическим ожиданиям
Отклонения от идеальной линии могут быть вызваны:
Разной длиной образца (patternSize)
Коллизиями хешей (редкими для правильно выбранного простого числа)
Накладными расходами JVM (сборка мусора, JIT-компиляция)

2. Количество итераций:

График итераций также демонстрирует линейную зависимость
Количество итераций немного больше, чем размер текста, так как:
Включает вычисление начального хеша
Включает пересчет хеша для каждого окна
Включает проверку символов при совпадении хешей

3. Различия с идеальным случаем:

Идеальная сложность O(n) достигается, когда:
Нет коллизий хешей (не требуется проверка символов)
Длина образца мала по сравнению с текстом (пересчет хеша занимает мало времени)
В реальности всегда есть дополнительные операции, которые увеличивают время
