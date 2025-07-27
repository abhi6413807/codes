import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.util.HashMap;
import java.awt.print.*;

public class SupermarketBillingSystem extends JFrame implements ActionListener {

    // Components
    JComboBox<String> itemComboBox;
    JTextField quantityField, newItemField, newPriceField;
    JTextArea billArea;
    JButton addButton, clearButton, addItemButton, totalButton, saveButton, printButton;

    HashMap<String, Double> priceMap = new HashMap<>();
    double total = 0;

    public SupermarketBillingSystem() {
        setTitle("Supermarket Billing System (₹)");
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setSize(700, 700);
        setLocationRelativeTo(null);

        // ---------- DATA ----------
        priceMap.put("Apples", 40.0);
        priceMap.put("Bananas", 30.0);
        priceMap.put("Milk", 50.0);
        priceMap.put("Bread", 25.0);
        priceMap.put("Eggs", 60.0);

        // ---------- MAIN PANEL ----------
        JPanel mainPanel = new JPanel(new BorderLayout(10, 10));
        mainPanel.setBackground(new Color(245, 245, 245));
        mainPanel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));
        setContentPane(mainPanel);

        // ---------- TOP: BILL DISPLAY ----------
        billArea = new JTextArea();
        billArea.setEditable(false);
        billArea.setFont(new Font("Monospaced", Font.PLAIN, 14));
        billArea.append("====== Supermarket Bill (₹) ======\n");
        billArea.append(String.format("%-15s%-10s%-10s\n", "Item", "Qty", "Price"));
        JScrollPane scroll = new JScrollPane(billArea);
        scroll.setPreferredSize(new Dimension(660, 300));
        mainPanel.add(scroll, BorderLayout.NORTH);

        // ---------- CENTER: ACTION PANELS ----------
        JPanel centerPanel = new JPanel(new GridLayout(1, 2, 10, 10));
        centerPanel.setOpaque(false);
        mainPanel.add(centerPanel, BorderLayout.CENTER);

        // ---- Left: Add to Bill / Controls ----
        JPanel left = new JPanel(new GridBagLayout());
        left.setBackground(new Color(230, 230, 250));
        left.setBorder(BorderFactory.createTitledBorder("Billing Controls"));
        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(8, 8, 8, 8);
        c.fill = GridBagConstraints.HORIZONTAL;

        // Row 0: Item + Qty
        c.gridx = 0; c.gridy = 0;
        left.add(new JLabel("Select Item:"), c);
        itemComboBox = new JComboBox<>(priceMap.keySet().toArray(new String[0]));
        c.gridx = 1; 
        left.add(itemComboBox, c);

        c.gridx = 0; c.gridy = 1;
        left.add(new JLabel("Quantity:"), c);
        quantityField = new JTextField();
        c.gridx = 1;
        left.add(quantityField, c);

        // Row 2: Add / Clear / Total
        addButton = new JButton("Add to Bill");
        clearButton = new JButton("Clear");
        totalButton = new JButton("Show Total");
        addButton.addActionListener(this);
        clearButton.addActionListener(this);
        totalButton.addActionListener(this);

        JPanel btnRow = new JPanel(new FlowLayout(FlowLayout.CENTER, 10, 0));
        btnRow.setOpaque(false);
        btnRow.add(addButton);
        btnRow.add(totalButton);
        btnRow.add(clearButton);
        c.gridx = 0; c.gridy = 2; c.gridwidth = 2;
        left.add(btnRow, c);

        centerPanel.add(left);

        // ---- Right: Add New Item + Save/Print ----
        JPanel right = new JPanel(new GridBagLayout());
        right.setBackground(new Color(250, 240, 230));
        right.setBorder(BorderFactory.createTitledBorder("Settings & Output"));
        GridBagConstraints d = new GridBagConstraints();
        d.insets = new Insets(8, 8, 8, 8);
        d.fill = GridBagConstraints.HORIZONTAL;

        // New Item Form
        d.gridx = 0; d.gridy = 0;
        right.add(new JLabel("New Item Name:"), d);
        newItemField = new JTextField();
        d.gridx = 1;
        right.add(newItemField, d);

        d.gridx = 0; d.gridy = 1;
        right.add(new JLabel("New Price (₹):"), d);
        newPriceField = new JTextField();
        d.gridx = 1;
        right.add(newPriceField, d);

        addItemButton = new JButton("Add New Item");
        addItemButton.addActionListener(this);
        d.gridx = 0; d.gridy = 2; d.gridwidth = 2;
        right.add(addItemButton, d);

        // Save & Print Buttons
        saveButton = new JButton("Save Bill");
        printButton = new JButton("Print Bill");
        saveButton.addActionListener(this);
        printButton.addActionListener(this);
        JPanel outRow = new JPanel(new FlowLayout(FlowLayout.CENTER, 10, 0));
        outRow.setOpaque(false);
        outRow.add(saveButton);
        outRow.add(printButton);
        d.gridx = 0; d.gridy = 3; d.gridwidth = 2;
        right.add(outRow, d);

        centerPanel.add(right);

        pack();
        setVisible(true);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == addButton) {
            String item = (String) itemComboBox.getSelectedItem();
            try {
                int qty = Integer.parseInt(quantityField.getText().trim());
                if (qty <= 0) throw new NumberFormatException();
                double price = priceMap.get(item);
                double sub = price * qty;
                total += sub;
                billArea.append(String.format("%-15s%-10d₹%-10.2f\n", item, qty, sub));
                quantityField.setText("");
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(this, "Enter valid quantity!");
            }
        }
        else if (e.getSource() == totalButton) {
            billArea.append("\n--------------------------------\n");
            billArea.append(String.format("Total Amount: ₹%.2f\n", total));
            totalButton.setEnabled(false);
        }
        else if (e.getSource() == clearButton) {
            billArea.setText("====== Supermarket Bill (₹) ======\n");
            billArea.append(String.format("%-15s%-10s%-10s\n", "Item", "Qty", "Price"));
            total = 0;
            totalButton.setEnabled(true);
        }
        else if (e.getSource() == addItemButton) {
            String name = newItemField.getText().trim();
            String pr = newPriceField.getText().trim();
            if (name.isEmpty() || pr.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Enter name & price!");
                return;
            }
            try {
                double p = Double.parseDouble(pr);
                if (p <= 0) throw new NumberFormatException();
                priceMap.put(name, p);
                itemComboBox.addItem(name);
                JOptionPane.showMessageDialog(this, "Item added.");
                newItemField.setText("");
                newPriceField.setText("");
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(this, "Enter valid price!");
            }
        }
        else if (e.getSource() == saveButton) {
            try {
                JFileChooser fc = new JFileChooser();
                if (fc.showSaveDialog(this) == JFileChooser.APPROVE_OPTION) {
                    FileWriter w = new FileWriter(fc.getSelectedFile());
                    w.write(billArea.getText());
                    w.close();
                    JOptionPane.showMessageDialog(this, "Saved ✓");
                }
            } catch (IOException ex) {
                JOptionPane.showMessageDialog(this, "Save error!");
            }
        }
        else if (e.getSource() == printButton) {
            try {
                billArea.print();
            } catch (PrinterException ex) {
                JOptionPane.showMessageDialog(this, "Print error!");
            }
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(SupermarketBillingSystem::new);
    }
}
