
## Clase `Exchanger<V>`

**Punto de sincronización donde se intercambian objetos entre dos hilos**
Lo que se pasa por el operador diamante es el tipo de objeto que se quiere compartir entre los hilos.

Tenemos dos métodos:
`exchange(V x)`
`exchange(V x, long timeout, TimeUnit timeUnit`

El hilo que quiere obtener información espera realizando llamada al método `exchange` hasta que el otro hilo sitúe la información usando el mismo método o hasta que pase el tiempo.
Ambos hilos intercambiarán objetos del mismo tipo.

**Funcionamiento**
- HiloA invoca a `exchange(objetoA)` e hiloB incova a `exchange(objectoB)`
- El hilo que procesa su llamada en primer lugar se bloquea y queda a la espera de que lo haga el segundo.
- Cuando eso sucede, se libera el bloqueo sobre ambos hilos y la salida de `exchange(objetoA)` da objetoB  y la salida de `exchange(objetoB)` da objeto A. 

**Ejemplo**
Tendremos dos hilos: uno será el productor que generará datos y el otro será el consumidor que los consumirá. Utilizaremos la clase `Exchanger` para que el productor envíe los datos al consumidor y viceversa.

```java
import java.util.concurrent.Exchanger;

class ProducerConsumerExample {
    public static void main(String[] args) {
        Exchanger<String> exchanger = new Exchanger<>();

        // Hilo Productor
        Thread producerThread = new Thread(() -> {
            try {
                String dataProduced = "Data from Producer";
                System.out.println("Producer produced: " + dataProduced);

                // Intercambia datos con el consumidor
                String dataReceived = exchanger.exchange(dataProduced);

                System.out.println("Producer received: " + dataReceived);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // Hilo Consumidor
        Thread consumerThread = new Thread(() -> {
            try {
                String dataConsumed = "Data from Consumer";
                System.out.println("Consumer produced: " + dataConsumed);

                // Intercambia datos con el productor
                String dataReceived = exchanger.exchange(dataConsumed);

                System.out.println("Consumer received: " + dataReceived);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        producerThread.start();
        consumerThread.start();
    }
}
```



## Clase `CountDownLatch`

Permite que uno o más hilos esperen hasta que los otros finalicen su trabajo.

**Dinámica**
- Se implementa un punto de espera (puerta de cierre) donde uno o más hilos esperan a que otros finalicen su trabajo
- Los hilos que finalicen se controlan por un contador (countdown, cuenta atrás)
- Cuando la cuenta atrás llegue a cero se reanudará el trabajo del hilo/hilos interrumpidos
- La cuenta atrás no puede reiniciarse 

Está el constructor
```java
countDownLatch(int cuenta)
```

El hilo en curso invoca al método **await()** y espera en la puerta de cierre.
También está **await(long tiempoespera, TimeUnit unit)** para indicar que no espere más si se sobrepasa el tiempo.

La cuenta atrás se decrementa con **countDown()** 

Método **getCount()** obtiene el valor de la cuenta atrás.

**Ejemplo**

Tres trabajadores completen sus tareas antes de que el supervisor consolide los resultados y genere informe final. 

```java
import java.util.concurrent.CountDownLatch;

class Worker implements Runnable {
    private final CountDownLatch latch;
    private final String name;

    public Worker(CountDownLatch latch, String name) {
        this.latch = latch;
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println("Worker " + name + " is working...");
        try {
            Thread.sleep(2000); // Simulamos que el trabajador está trabajando durante 2 segundos
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Worker " + name + " has completed its task.");
        latch.countDown(); // Señalamos que el trabajador ha completado su tarea
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(3); // Contador inicializado en 3

        // Creamos e iniciamos los trabajadores
        Thread worker1 = new Thread(new Worker(latch, "1"));
        Thread worker2 = new Thread(new Worker(latch, "2"));
        Thread worker3 = new Thread(new Worker(latch, "3"));
        worker1.start();
        worker2.start();
        worker3.start();

        latch.await(); // Esperamos a que todos los trabajadores completen sus tareas
        System.out.println("All workers have completed their tasks. Supervisor can now consolidate results.");
    }
}

```

## Clase `CyclicBarrier`

Permite que uno o más threads se esperen hasta que todos ellos finalicen su trabajo.
Se diferencia de `CountDownLatch` principalmente en que esta cuenta **se puede reiniciar**, **volver a usar**

Se implementa un punto de espera llamado barrera.

Al constructor `CyclicBarrier(int hilos)` se le indican los hilos que usaran la barrera. Este número disparará la barrera cunado a ella llegue el último hilo. 

Es posible lanzar una acción cuando los hilos lleguen a la barrera `CyclicBarrier(int hilos, Runnable accion)` 

- El método **await()** es el que se usa para indicar que el hilo en curso ha concluido su trabajo y está esperando que lleguen los demás.  Es la barrera.
- **await(long tiempo, TimeUnit unit)** marca límite en la espera
- **reset()** reiniciar barrera
- **getNumberWaiting()** Número de hilos en espera
- **getParties()** Número de hilos requeridos por la barrera

**Ejemplo**
Supongamos que tenemos una carrera con tres corredores, y queremos que todos se preparen antes de comenzar la carrera.

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

class Runner implements Runnable {
    private final CyclicBarrier barrier;
    private final String name;

    public Runner(CyclicBarrier barrier, String name) {
        this.barrier = barrier;
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println(name + " is getting ready...");
        try {
            Thread.sleep(2000); // Simulamos que el corredor se está preparando durante 2 segundos
            System.out.println(name + " is ready!");
            barrier.await(); // Esperamos a que todos los corredores estén listos
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
        System.out.println(name + " started running.");
    }
}

public class Main {
    public static void main(String[] args) {
        CyclicBarrier barrier = new CyclicBarrier(3); // CyclicBarrier con 3 participantes

        // Creamos e iniciamos los corredores
        Thread runner1 = new Thread(new Runner(barrier, "Runner 1"));
        Thread runner2 = new Thread(new Runner(barrier, "Runner 2"));
        Thread runner3 = new Thread(new Runner(barrier, "Runner 3"));
        runner1.start();
        runner2.start();
        runner3.start();
    }
}

```


## Interfaz `Executor` y pools de hilos

En **aplicaciones tipo servidor** estas deben atender número masivo y concurrente de peticiones de usuario. 
Ejecutar cada tarea en hilo distinto puede llegar a crear tantos hilos que se compromete la estabilidad del sistema.

Lo ideal es un pool de hilos. 
Es un contenedor en el que se crean y se inician un número limitado de hilos para ejecutar todas las tareas de una lista. 

Lo normal será que sea un objeto `ExecutorService` usando algún método de la clase estática `Executors`:

- **newFixedThreadPool(int numberThreads)**: Pool con el número de hilos indicado. Se reutilizarán cíclicamente hasta terminar las tareas de la cola.
- **newCachedThreadPool()**: Crea hilos conforme se necesiten, reutilizando los ya concluidos para no crear demasiados. Hilos inactivos son terminados automáticamente por el pool
- **newSingleThreadExecutor()**: Pool de un solo hilo. La excepción en una tarea no detendrá la ejecución de las siguientes. 
- **newScheduledExecutor()**: Pool que ejecuta tareas programadas cada cierto tiempo. De una sola vez o de ofrma repetitiva. Se parece al `Timer` pero puede tener varios threads que iran realizando las tareas programadas.


Los `ExecutorService` implementan `Executor` que tiene el método **execute(Runnable)** , que es el que se llama por cada tarea que deba ejecutarse en el pool.

También está el método **shutdown()** para indicar al pool que los hilos deben morir cuando finalicen su trabajo; no serán utilizados para nuevas tareas. 

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class Task implements Runnable {
    private final int taskId;

    public Task(int taskId) {
        this.taskId = taskId;
    }

    @Override
    public void run() {
        System.out.println("Task " + taskId + " is executing by thread: " + Thread.currentThread().getName());
        // Simulamos alguna tarea que toma un tiempo aleatorio
        try {
            Thread.sleep((long) (Math.random() * 1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Task " + taskId + " completed.");
    }
}

public class Main {
    public static void main(String[] args) {
        // Creamos un ExecutorService con un conjunto de hilos fijos de tamaño 3
        ExecutorService executor = Executors.newFixedThreadPool(3);

        // Creamos e iniciamos varias tareas
        for (int i = 1; i <= 5; i++) {
            executor.execute(new Task(i));
        }

        // Cerramos el ExecutorService después de que todas las tareas han sido enviadas
        executor.shutdown();
    }
}
```

------------

Hay opción de usar `submit(Callable)`para enviar tareas con resultado y luego se recuperan los resultados usando `Future`

```java
import java.util.concurrent.*;

public class SubmitExample {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        // Enviar tareas usando submit() que devuelven un resultado
        Future<Integer> result1 = executor.submit(() -> {
            Thread.sleep(1000);
            return 1;
        });

        Future<Integer> result2 = executor.submit(() -> {
            Thread.sleep(2000);
            return 2;
        });

        Future<Integer> result3 = executor.submit(() -> {
            Thread.sleep(1500);
            return 3;
        });

        // Esperar y obtener los resultados
        int total = result1.get() + result2.get() + result3.get();
        System.out.println("Total: " + total);

        executor.shutdown();
    }
}
```

---------

#### Más ejemplos de ExecutorService 

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CachedThreadPoolExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();

        for (int i = 0; i < 5; i++) {
            final int taskId = i;
            executor.execute(() -> {
                System.out.println("Task " + taskId + " executed by thread: " + Thread.currentThread().getName());
            });
        }

        executor.shutdown();
    }
}

```

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SingleThreadExecutorExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newSingleThreadExecutor();

        for (int i = 0; i < 5; i++) {
            final int taskId = i;
            executor.execute(() -> {
                System.out.println("Task " + taskId + " executed by thread: " + Thread.currentThread().getName());
            });
        }

        executor.shutdown();
    }
}

```


```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class ScheduledThreadPoolExample {
    public static void main(String[] args) {
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

        Runnable task = () -> System.out.println("Tarea ejecutada");

        // Ejecutar la tarea después de un retraso inicial de 1 segundo
        executor.schedule(task, 1, TimeUnit.SECONDS);

        executor.shutdown();
    }
}
```


```
// Ejecutar la tarea después de un retraso inicial de 1 segundo, luego repetirla cada 3 segundos 
executor.scheduleAtFixedRate(task, 1, 3, TimeUnit.SECONDS);
```


Enviar correo todos los días a las 00:00 (ejemplos absurdos que se me ocurren)

```java
import java.time.LocalTime;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class ScheduledEmailSender {

    public static void main(String[] args) {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

        // Obtén la hora actual
        LocalTime now = LocalTime.now();
        // Calcula la cantidad de tiempo hasta la próxima medianoche
        long initialDelay = LocalTime.MIDNIGHT.toSecondOfDay() - now.toSecondOfDay();

        // Programa el envío del correo electrónico todos los días a las 00:00
        scheduler.scheduleAtFixedRate(EmailSender::sendEmail, initialDelay, TimeUnit.DAYS.toSeconds(1), TimeUnit.SECONDS);
    }
}

```

----------

### Callable y Future

Como viste, `Callable` se usa con `ExecutorService` para enviar tareas en las que se quiere obtener un resultado. Este es devuelto como `Future` y se puede hacer un `get()`

### Timer

Se usa para programar tareas para ejecutarse en el futuro de forma repetitiva o una sola vez. Pero no es seguro en entorno concurrente. 

```java

public class TimerExample {
    public static void main(String[] args) {
        Timer timer = new Timer();
        timer.schedule(new MyTask(), 1000, 2000);
    }

    static class MyTask extends TimerTask {
        @Override
        public void run() {
            System.out.println("Tarea ejecutada en el tiempo actual: " + System.currentTimeMillis());
        }
    }
}
```

En este ejemplo, la tarea `MyTask` se ejecutará cada 2 segundos después de un retraso inicial de 1 segundo.

Podríamos abortar pasado un tiempo, por ejemplo:

```java
@Stateless
public class RequestHandler {

    @Resource
    TimerService timerService;

    public void processRequest() {
        // Establecer un temporizador para la solicitud
        timerService.createSingleActionTimer(10000, new TimerConfig());
    }

    @Timeout
    public void handleTimeout(Timer timer) {
        // Abortar la solicitud cuando se alcance el tiempo límite
        System.out.println("La solicitud ha excedido el límite de tiempo. Abortando la solicitud...");
        // Aquí puedes incluir el código para abortar la solicitud
    }
}
```

----------

Aunque no interesa mucho, JavaEE permite programar 

```java
import javax.ejb.Schedule;
import javax.ejb.Singleton;

@Singleton
public class ExportScheduler {

    @Schedule(hour = "0", minute = "0", second = "0", persistent = false)
    public void exportFile() {
        // Lógica para generar y enviar el archivo de exportación
        // Puedes llamar a métodos de otras clases aquí o incluir la lógica directamente en este método
        // Por ejemplo, podrías llamar a un método de una clase dedicada a la generación de archivos de envio
    }
}
```

-------

Jasper

```java
import net.sf.jasperreports.engine.*;
import net.sf.jasperreports.engine.util.JRLoader;

import java.io.File;
import java.util.Map;

public class ReportTask implements Runnable {
    private final String templatePath;
    private final Map<String, Object> parameters;
    private final JRDataSource dataSource;
    private final String outputPath;

    public ReportTask(String templatePath, Map<String, Object> parameters, JRDataSource dataSource, String outputPath) {
        this.templatePath = templatePath;
        this.parameters = parameters;
        this.dataSource = dataSource;
        this.outputPath = outputPath;
    }

    @Override
    public void run() {
        try {
            JasperReport jasperReport = (JasperReport) JRLoader.loadObject(new File(templatePath));
            JasperPrint jasperPrint = JasperFillManager.fillReport(jasperReport, parameters, dataSource);
            JasperExportManager.exportReportToPdfFile(jasperPrint, outputPath);
        } catch (JRException e) {
            e.printStackTrace();
        }
    }


```


```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ParallelReportGenerator {
    public static void main(String[] args) {
        // Configuración del ExecutorService
        int numberOfThreads = 4;  // Ajusta este valor según el hardware disponible
        ExecutorService executorService = Executors.newFixedThreadPool(numberOfThreads);

        // Supongamos que tienes una lista de registros que necesitas procesar
        List<Record> records = getRecordsToProcess();

        // Genera informes en paralelo
        for (Record record : records) {
            // Configura los parámetros y dataSource para cada registro
            Map<String, Object> parameters = getParametersForRecord(record);
            JRDataSource dataSource = getDataSourceForRecord(record);
            String outputPath = getOutputPathForRecord(record);

            // Crea y envía la tarea al ExecutorService
            ReportTask task = new ReportTask("path/to/template.jasper", parameters, dataSource, outputPath);
            executorService.submit(task);
        }

        // Cierra el ExecutorService
        executorService.shutdown();
        try {
            // Espera a que todas las tareas finalicen
            if (!executorService.awaitTermination(60, TimeUnit.MINUTES)) {
                executorService.shutdownNow();
            }
        } catch (InterruptedException e) {
            executorService.shutdownNow();
        }
    }

    // Métodos para obtener registros, parámetros, dataSource y ruta de salida (implementación omitida)
    private static List<Record> getRecordsToProcess() {
        // ...
    }

    private static Map<String, Object> getParametersForRecord(Record record) {
        // ...
    }

    private static JRDataSource getDataSourceForRecord(Record record) {
        // ...
    }

    private static String getOutputPathForRecord(Record record) {
        // ...
    }
}

```

```java
import net.sf.jasperreports.engine.JasperPrint;
import net.sf.jasperreports.engine.util.JRLoader;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class CombineReports {
    public static void main(String[] args) {
        List<JasperPrint> jasperPrints = new ArrayList<>();

        // Cargar cada archivo JasperPrint generado
        for (String outputPath : getGeneratedOutputPaths()) {
            try {
                JasperPrint jasperPrint = (JasperPrint) JRLoader.loadObject(new File(outputPath));
                jasperPrints.add(jasperPrint);
            } catch (JRException e) {
                e.printStackTrace();
            }
        }

        // Combina los JasperPrints en uno solo
        JasperPrint combinedJasperPrint = new JasperPrint();
        for (JasperPrint jasperPrint : jasperPrints) {
            combinedJasperPrint.getPages().addAll(jasperPrint.getPages());
        }

        // Exporta el documento combinado a PDF
        try {
            JasperExportManager.exportReportToPdfFile(combinedJasperPrint, "path/to/combinedReport.pdf");
        } catch (JRException e) {
            e.printStackTrace();
        }
    }

    // Método para obtener las rutas de salida generadas (implementación omitida)
    private static List<String> getGeneratedOutputPaths() {
        // ...
    }
}

```


------

Uso de `ExecutorService` en JasperReport para agilizar producción de informes (Pregunté a ChatGPT)

```java
import java.util.ArrayList;
import java.util.List;

public class ListUtils {

    public static <T> List<List<T>> partition(List<T> list, int partitions) {
        List<List<T>> subLists = new ArrayList<>();
        int totalSize = list.size();
        int subListSize = (int) Math.ceil((double) totalSize / partitions);

        for (int i = 0; i < totalSize; i += subListSize) {
            subLists.add(new ArrayList<T>(list.subList(i, Math.min(totalSize, i + subListSize))));
        }

        return subLists;
    }
}

```

```java
import net.sf.jasperreports.engine.*;
import net.sf.jasperreports.engine.util.JRLoader;
import com.itextpdf.text.Document;
import com.itextpdf.text.pdf.PdfCopy;
import com.itextpdf.text.pdf.PdfReader;
import com.itextpdf.text.pdf.PdfWriter;

import javax.annotation.PostConstruct;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.ViewScoped;
import javax.faces.context.FacesContext;
import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.*;

@ManagedBean
@ViewScoped
public class ReportBean implements Serializable {
    private static final long serialVersionUID = 1L;

    private ExecutorService executorService;
    private List<Record> records;

    @PostConstruct
    public void init() {
        // Inicializar la lista de registros a procesar
        records = getRecordsToProcess();
        // Inicializar el ExecutorService con un número adecuado de hilos
        int numberOfThreads = 4;  // Ajustar según el hardware disponible
        executorService = Executors.newFixedThreadPool(numberOfThreads);
    }

    public void generateAndDownloadReports() {
        try {
            // Dividir la lista de registros en 4 sublistas
            List<List<Record>> subLists = ListUtils.partition(records, 4);

            // Lista para almacenar los futuros de los informes generados
            List<Future<List<byte[]>>> futures = new ArrayList<>();

            // Enviar una tarea por cada sublista
            for (final List<Record> subList : subLists) {
                Future<List<byte[]>> future = executorService.submit(new Callable<List<byte[]>>() {
                    @Override
                    public List<byte[]> call() throws Exception {
                        List<byte[]> reportBytesList = new ArrayList<>();
                        for (Record record : subList) {
                            ByteArrayOutputStream baos = new ByteArrayOutputStream();
                            try {
                                Map<String, Object> parameters = getParametersForRecord(record);
                                JRDataSource dataSource = getDataSourceForRecord(record);

                                JasperReport jasperReport = (JasperReport) JRLoader.loadObject(new File("path/to/template.jasper"));
                                JasperPrint jasperPrint = JasperFillManager.fillReport(jasperReport, parameters, dataSource);
                                JasperExportManager.exportReportToPdfStream(jasperPrint, baos);
                            } catch (JRException e) {
                                e.printStackTrace();
                            }
                            reportBytesList.add(baos.toByteArray());
                        }
                        return reportBytesList;
                    }
                });
                futures.add(future);
            }

            // Esperar a que todos los informes se generen y combinar los resultados
            List<byte[]> reports = new ArrayList<>();
            for (Future<List<byte[]>> future : futures) {
                reports.addAll(future.get());
            }

            // Combinar todos los informes en un solo archivo PDF
            ByteArrayOutputStream combinedBaos = new ByteArrayOutputStream();
            combinePDFs(reports, combinedBaos);

            // Enviar el informe combinado en la respuesta HTTP para su descarga
            FacesContext facesContext = FacesContext.getCurrentInstance();
            HttpServletResponse response = (HttpServletResponse) facesContext.getExternalContext().getResponse();
            response.reset();
            response.setContentType("application/pdf");
            response.setHeader("Content-Disposition", "attachment; filename=\"combined_report.pdf\"");

            OutputStream output = response.getOutputStream();
            output.write(combinedBaos.toByteArray());
            output.flush();
            output.close();

            facesContext.responseComplete();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            executorService.shutdown();
        }
    }

    private void combinePDFs(List<byte[]> pdfs, OutputStream outputStream) throws IOException, DocumentException {
        Document document = new Document();
        PdfCopy copy = new PdfCopy(document, outputStream);
        document.open();
        for (byte[] pdf : pdfs) {
            PdfReader reader = new PdfReader(pdf);
            int n = reader.getNumberOfPages();
            for (int page = 0; page < n;) {
                copy.addPage(copy.getImportedPage(reader, ++page));
            }
            reader.close();
        }
        document.close();
    }

    // Métodos para obtener registros, parámetros y dataSource (implementación omitida)
    private List<Record> getRecordsToProcess() {
        // ...
    }

    private Map<String, Object> getParametersForRecord(Record record) {
        // ...
    }

    private JRDataSource getDataSourceForRecord(Record record) {
        // ...
    }
}

```

```java
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://xmlns.jcp.org/jsf/core">
<h:head>
    <title>Generación de Informes</title>
</h:head>
<h:body>
    <h:form>
        <h:commandButton value="Generar y Descargar Informes" action="#{reportBean.generateAndDownloadReports}" />
    </h:form>
</h:body>
</html>

```

