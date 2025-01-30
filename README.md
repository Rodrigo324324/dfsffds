import javafx.application.Application;
import javafx.geometry.Insets;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.layout.*;
import javafx.scene.paint.Color;
import javafx.scene.text.Font;
import javafx.stage.Stage;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.PDPageContentStream;
import org.apache.pdfbox.pdmodel.font.PDType1Font;
import java.sql.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Optional;

public class Main extends Application {

    // Colores y estilo
    private static final String COLOR_FONDO = "#d4edda";
    private static final String COLOR_BOTONES = "#28a745";
    private static final String COLOR_TITULOS = "#155724";

    private Connection conexion;
    private TabPane tabPane;

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        conectarBD();
        crearTablas();

        // Configuración principal
        BorderPane root = new BorderPane();
        root.setTop(crearBanner());
        tabPane = new TabPane();
        tabPane.getTabs().addAll(
                crearTabClientes(),
                crearTabPresupuestos(),
                crearTabVisitas(),
                crearTabGanancias()
        );
        root.setCenter(tabPane);

        Scene scene = new Scene(root, 1000, 600);
        scene.getStylesheets().add(getClass().getResource("styles.css").toExternalForm());
        primaryStage.setTitle("🌿 Gestión de Jardinería");
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    private HBox crearBanner() {
        HBox banner = new HBox();
        banner.setStyle("-fx-background-color: " + COLOR_TITULOS + ";");
        Label titulo = new Label("  Sistema de Gestión de Jardinería Profesional");
        titulo.setFont(Font.font("Arial", 24));
        titulo.setTextFill(Color.WHITE);
        banner.getChildren().add(titulo);
        banner.setPadding(new Insets(15));
        return banner;
    }

    private Tab crearTabClientes() {
        Tab tab = new Tab("🌱 Clientes");
        GridPane grid = new GridPane();
        grid.setPadding(new Insets(20));
        grid.setHgap(10);
        grid.setVgap(10);

        // Campos del formulario
        TextField nombre = new TextField();
        TextField telefono = new TextField();
        TextField email = new TextField();
        TextField direccion = new TextField();

        grid.add(new Label("Nombre:"), 0, 0);
        grid.add(nombre, 1, 0);
        grid.add(new Label("Teléfono:"), 0, 1);
        grid.add(telefono, 1, 1);
        grid.add(new Label("Email:"), 0, 2);
        grid.add(email, 1, 2);
        grid.add(new Label("Dirección:"), 0, 3);
        grid.add(direccion, 1, 3);

        Button btnGuardar = new Button("Guardar Cliente");
        btnGuardar.setStyle("-fx-background-color: " + COLOR_BOTONES + "; -fx-text-fill: white;");
        btnGuardar.setOnAction(e -> guardarCliente(nombre.getText(), telefono.getText(), email.getText(), direccion.getText()));

        grid.add(btnGuardar, 1, 4);

        tab.setContent(grid);
        return tab;
    }

    private void guardarCliente(String nombre, String telefono, String email, String direccion) {
        String sql = "INSERT INTO clientes(nombre, telefono, email, direccion) VALUES(?,?,?,?)";
        try (PreparedStatement pstmt = conexion.prepareStatement(sql)) {
            pstmt.setString(1, nombre);
            pstmt.setString(2, telefono);
            pstmt.setString(3, email);
            pstmt.setString(4, direccion);
            pstmt.executeUpdate();
            mostrarAlerta("Cliente guardado exitosamente!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private Tab crearTabPresupuestos() {
        Tab tab = new Tab("💰 Presupuestos");
        VBox vbox = new VBox(10);
        vbox.setPadding(new Insets(20));

        // Formulario
        TextField servicios = new TextField();
        TextField total = new TextField();
        ComboBox<String> clientes = new ComboBox<>();

        Button btnGenerarPDF = new Button("Generar PDF");
        btnGenerarPDF.setStyle("-fx-background-color: " + COLOR_BOTONES + "; -fx-text-fill: white;");

        vbox.getChildren().addAll(
                new Label("Cliente:"), clientes,
                new Label("Servicios:"), servicios,
                new Label("Total:"), total,
                btnGenerarPDF
        );

        tab.setContent(vbox);
        return tab;
    }

    private void generarPDF(String cliente, String servicios, String total) {
        try (PDDocument doc = new PDDocument()) {
            PDPage page = new PDPage();
            doc.addPage(page);

            try (PDPageContentStream content = new PDPageContentStream(doc, page)) {
                content.beginText();
                content.setFont(PDType1Font.HELVETICA_BOLD, 18);
                content.newLineAtOffset(50, 700);
                content.showText("Presupuesto de Jardinería");
                content.endText();

                // Contenido dinámico...
            }

            doc.save("presupuesto_" + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss")) + ".pdf");
            mostrarAlerta("PDF generado exitosamente!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Métodos para Visitas y Ganancias (similar a los anteriores)

    private void conectarBD() {
        try {
            conexion = DriverManager.getConnection("jdbc:sqlite:jardineria.db");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void crearTablas() {
        String sqlClientes = "CREATE TABLE IF NOT EXISTS clientes (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT," +
                "nombre TEXT," +
                "telefono TEXT," +
                "email TEXT," +
                "direccion TEXT)";

        String sqlPresupuestos = "CREATE TABLE IF NOT EXISTS presupuestos (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT," +
                "cliente_id INTEGER," +
                "fecha TEXT," +
                "servicios TEXT," +
                "total REAL," +
                "estado TEXT)";

        ejecutarSQL(sqlClientes);
        ejecutarSQL(sqlPresupuestos);
    }

    private void ejecutarSQL(String sql) {
        try (Statement stmt = conexion.createStatement()) {
            stmt.execute(sql);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void mostrarAlerta(String mensaje) {
        Alert alert = new Alert(Alert.AlertType.INFORMATION);
        alert.setTitle("Operación Exitosa");
        alert.setHeaderText(null);
        alert.setContentText(mensaje);
        alert.showAndWait();
    }

    @Override
    public void stop() {
        try {
            if (conexion != null) conexion.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
