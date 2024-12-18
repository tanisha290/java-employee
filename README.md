//code in src folder

import java.io.*;
import java.util.*;
import java.util.concurrent.*;

import static java.util.concurrent.Executors.newFixedThreadPool;

class Employee {
    private int id;
    private String name;
    private double salary;

    public Employee(int id, String name, double salary) {
        this.id = id;
        this.name = name;
        this.salary = salary;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }

    @Override
    public String toString() {
        return id + ", " + name + ", " + salary;
    }
}

public class EmployeeFileHandler {
    public static void writeToFile(List<Employee> employees, String filename) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            for (Employee employee : employees) {
                writer.write(employee.toString());
                writer.newLine();
            }
        } catch (IOException exc) {
            System.err.println("Error writing to file " + filename + ": " + exc.getMessage());
            throw exc;
        }
    }

    public static List<String> readLines(String filename) throws IOException {
        List<String> lines = new ArrayList<>();
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
        }
        return lines;
    }

    public static void readFilesparallel(String file1, String file2) throws InterruptedException {
        ExecutorService exe = newFixedThreadPool(2);
        List<String> lines1 = Collections.synchronizedList(new ArrayList<>());
        List<String> lines2 = Collections.synchronizedList(new ArrayList<>());

        Runnable taskA = () -> {
            try {
                lines1.addAll(readLines(file1));
            } catch (IOException exc) {
                System.err.println("Error reading file " + file1 + ": " + exc.getMessage());
            }
        };

        Runnable taskB = () -> {
            try {
                lines2.addAll(readLines(file2));
            } catch (IOException exc) {
                System.err.println("Error reading file " + file2 + ": " + exc.getMessage());
            }
        };

        exe.execute(taskA);
        exe.execute(taskB);

        exe.shutdown();
        exe.awaitTermination(60, TimeUnit.SECONDS);



        int maxSize = Math.max(lines1.size(), lines2.size());
        for (int i = 0; i < maxSize; i++) {
            if (i < lines1.size()) {
                System.out.println(lines1.get(i));
            }
            if (i < lines2.size()) {
                System.out.println(lines2.get(i));
            }
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException {

        List<Employee> employees1 = Arrays.asList(
                new Employee(1, "Karan", 150000),
                new Employee(2, "Rohit", 60000),
                new Employee(3, "Ruchika", 185000)
        );

        List<Employee> employees2 = Arrays.asList(
                new Employee(4, "Daksh", 55330),
                new Employee(5, "Tanisha", 200000),
                new Employee(6, "Anika", 10000)
        );


        String file1 = "employees1.txt";
        String file2 = "employees2.txt";
        writeToFile(employees1, file1);
        writeToFile(employees2, file2);

        System.out.println("Interleaved Employee Data:");
        readFilesparallel(file1, file2);
    }
}
