# Railway-Reservation.
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;

class Passenger {
    String name;
    int age;
    String gender;
    int seatNumber;

    Passenger(String name, int age, String gender, int seatNumber) {
        this.name = name;
        this.age = age;
        this.gender = gender;
        this.seatNumber = seatNumber;
    }
}

class Train {
    String trainName;
    int totalSeats;
    boolean[] seatBooked;
    ArrayList<Passenger> passengers;

    Train(String trainName, int totalSeats) {
        this.trainName = trainName;
        this.totalSeats = totalSeats;
        seatBooked = new boolean[totalSeats + 1];
        passengers = new ArrayList<>();
    }

    boolean bookTicket(String name, int age, String gender) {
        for (int i = 1; i <= totalSeats; i++) {
            if (!seatBooked[i]) {
                seatBooked[i] = true;
                passengers.add(new Passenger(name, age, gender, i));
                return true;
            }
        }
        return false;
    }

    boolean cancelTicket(int seatNo) {
        for (int i = 0; i < passengers.size(); i++) {
            if (passengers.get(i).seatNumber == seatNo) {
                passengers.remove(i);
                seatBooked[seatNo] = false;
                return true;
            }
        }
        return false;
    }

    Object[][] getPassengerData() {
        Object[][] data = new Object[passengers.size()][4];
        for (int i = 0; i < passengers.size(); i++) {
            Passenger p = passengers.get(i);
            data[i][0] = p.name;
            data[i][1] = p.age;
            data[i][2] = p.gender;
            data[i][3] = p.seatNumber;
        }
        return data;
    }

    Object[][] getAvailableSeats() {
        ArrayList<Integer> available = new ArrayList<>();
        for (int i = 1; i <= totalSeats; i++) {
            if (!seatBooked[i]) available.add(i);
        }
        Object[][] data = new Object[available.size()][1];
        for (int i = 0; i < available.size(); i++) {
            data[i][0] = available.get(i);
        }
        return data;
    }
}

public class RailwayReservationGUI extends JFrame {
    Train train;

    public RailwayReservationGUI() {
        train = new Train("Chennai Express", 10);
        setTitle(" Railway Reservation System");
        setSize(900, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        JTabbedPane tabbedPane = new JTabbedPane();

        // Home Tab
        JPanel homePanel = new JPanel(new BorderLayout());
        JLabel welcomeLabel = new JLabel("Welcome to Railway Reservation System", JLabel.CENTER);
        welcomeLabel.setFont(new Font("Segoe UI", Font.BOLD, 22));
        homePanel.add(welcomeLabel, BorderLayout.CENTER);
        tabbedPane.add(" Home", homePanel);

        // Timetable Tab
        tabbedPane.add("Timetable", createTimetablePanel());

        // Booking Tab
        tabbedPane.add(" Book Ticket", createBookingPanel());

        // View Bookings Tab
        tabbedPane.add(" View Bookings", createViewPanel());

        // Cancel Tab
        tabbedPane.add(" Cancel Ticket", createCancelPanel());

        // Available Seats
        tabbedPane.add(" Available Seats", createAvailableSeatsPanel());

        add(tabbedPane);
    }

    private JPanel createTimetablePanel() {
        String[] columns = {"Train No", "Train Name", "Departure", "Arrival", "From", "To", "Days"};
        Object[][] data = {
            {"12623", "Chennai Express", "06:00 AM", "12:30 PM", "Chennai", "Bangalore", "Mon, Wed, Fri"},
            {"12711", "Pinakini Express", "08:30 AM", "02:45 PM", "Chennai", "Vijayawada", "Daily"},
            {"12540", "Brindavan Express", "07:50 AM", "10:30 AM", "Bangalore", "Chennai", "Daily"},
            {"12864", "Howrah Express", "04:00 PM", "08:00 AM", "Chennai", "Howrah", "Tue, Thu, Sat"},
            {"12615", "Grand Trunk Express", "07:00 PM", "06:00 AM", "Delhi", "Chennai", "Daily"}
        };
        JTable table = new JTable(data, columns);
        table.setFont(new Font("Segoe UI", Font.PLAIN, 14));
        table.setRowHeight(25);
        JScrollPane scrollPane = new JScrollPane(table);

        JPanel panel = new JPanel(new BorderLayout());
        panel.add(scrollPane, BorderLayout.CENTER);
        return panel;
    }

    private JPanel createBookingPanel() {
        JPanel panel = new JPanel(new GridLayout(5, 2, 10, 10));
        panel.setBorder(BorderFactory.createEmptyBorder(20, 200, 20, 200));

        JLabel nameLabel = new JLabel("Name:");
        JLabel ageLabel = new JLabel("Age:");
        JLabel genderLabel = new JLabel("Gender:");

        JTextField nameField = new JTextField();
        JTextField ageField = new JTextField();
        JComboBox<String> genderBox = new JComboBox<>(new String[]{"Male", "Female", "Other"});

        JButton bookButton = new JButton("Book Ticket");
        JLabel statusLabel = new JLabel("", JLabel.CENTER);

        panel.add(nameLabel);
        panel.add(nameField);
        panel.add(ageLabel);
        panel.add(ageField);
        panel.add(genderLabel);
        panel.add(genderBox);
        panel.add(new JLabel(""));
        panel.add(bookButton);
        panel.add(statusLabel);

        bookButton.addActionListener(e -> {
            String name = nameField.getText();
            int age;
            try {
                age = Integer.parseInt(ageField.getText());
            } catch (Exception ex) {
                statusLabel.setText("⚠ Please enter valid age!");
                return;
            }
            String gender = genderBox.getSelectedItem().toString();

            boolean success = train.bookTicket(name, age, gender);
            if (success) {
                statusLabel.setText(" Ticket booked successfully!");
            } else {
                statusLabel.setText(" No seats available!");
            }
        });

        return panel;
    }

    private JPanel createViewPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        JButton refreshButton = new JButton("Refresh List");

        String[] columns = {"Name", "Age", "Gender", "Seat No"};
        DefaultTableModel model = new DefaultTableModel(columns, 0);
        JTable table = new JTable(model);
        table.setFont(new Font("Segoe UI", Font.PLAIN, 14));
        table.setRowHeight(25);
        JScrollPane scrollPane = new JScrollPane(table);

        refreshButton.addActionListener(e -> {
            model.setRowCount(0);
            Object[][] data = train.getPassengerData();
            for (Object[] row : data) model.addRow(row);
        });

        panel.add(refreshButton, BorderLayout.NORTH);
        panel.add(scrollPane, BorderLayout.CENTER);
        return panel;
    }

    private JPanel createCancelPanel() {
        JPanel panel = new JPanel(new GridLayout(2, 2, 10, 10));
        panel.setBorder(BorderFactory.createEmptyBorder(50, 250, 50, 250));

        JLabel seatLabel = new JLabel("Enter Seat Number:");
        JTextField seatField = new JTextField();
        JButton cancelButton = new JButton("Cancel Ticket");
        JLabel resultLabel = new JLabel("", JLabel.CENTER);

        panel.add(seatLabel);
        panel.add(seatField);
        panel.add(cancelButton);
        panel.add(resultLabel);

        cancelButton.addActionListener(e -> {
            try {
                int seatNo = Integer.parseInt(seatField.getText());
                boolean success = train.cancelTicket(seatNo);
                resultLabel.setText(success ? "Ticket cancelled!" : " Seat not found!");
            } catch (Exception ex) {
                resultLabel.setText("⚠ Invalid seat number!");
            }
        });

        return panel;
    }

    private JPanel createAvailableSeatsPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        JButton refreshButton = new JButton(" Refresh Seats");

        String[] columns = {"Available Seats"};
        DefaultTableModel model = new DefaultTableModel(columns, 0);
        JTable table = new JTable(model);
        table.setFont(new Font("Segoe UI", Font.PLAIN, 14));
        table.setRowHeight(25);
        JScrollPane scrollPane = new JScrollPane(table);

        refreshButton.addActionListener(e -> {
            model.setRowCount(0);
            Object[][] data = train.getAvailableSeats();
            for (Object[] row : data) model.addRow(row);
        });

        panel.add(refreshButton, BorderLayout.NORTH);
        panel.add(scrollPane, BorderLayout.CENTER);
        return panel;
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new RailwayReservationGUI().setVisible(true));
    }
}
