import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.ArrayList;

public class ChatServer extends JFrame implements ActionListener
{
    private JTextArea display;
    private JButton startButton;
    private JButton exitButton;
    private static JPanel buttonPanel;
    private static ServerSocket serverSocket;
    private static final int PORT = 1234;
    private ArrayList<ClientHandler> threads;

    public static void main(String[] args)
    {
        ChatServer frame = new ChatServer();
        frame.setSize(400, 300);
        frame.setVisible(true);
        frame.threads = new ArrayList<>();

        try
        {
            System.out.println("Port Opened");
            serverSocket = new ServerSocket(PORT);
        }
        catch (IOException ioEx)
        {
            System.out.print("Unable to attach to port!");
            System.exit(1);
        }

        frame.addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent event) {
                try {
                    serverSocket.close();
                } catch (IOException ioEx) {
                    System.out.println("Unable to close the server socket!");
                }
                System.exit(0);
            }
        });
    }

    public ChatServer()
    {
        display = new JTextArea(10, 15);
        display.setWrapStyleWord(true);
        display.setLineWrap(true);

        add(new JScrollPane(display), BorderLayout.CENTER);

        buttonPanel = new JPanel();
        startButton = new JButton("Start Server");
        startButton.addActionListener(this);
        buttonPanel.add(startButton);

        exitButton = new JButton("Exit");
        exitButton.addActionListener(this);
        buttonPanel.add(exitButton);

        add(buttonPanel, BorderLayout.SOUTH);

    }

    public void actionPerformed(ActionEvent event)
    {
        if (event.getSource() == startButton)
        {
            new Thread(this::handleIncomingConnections).start();
            display.append("Server Started \n");
        }

        if (event.getSource() == exitButton)
        {
            System.exit(0);
        }
    }

    private void handleIncomingConnections()
    {
        try {
            while (true)
            {
                Socket link = serverSocket.accept();
                ClientHandler clientHandler = new ClientHandler(link, threads);
                threads.add(clientHandler);
                clientHandler.start();
            }
        } catch (IOException ioEx) {
            display.append("Error: Unable to Handle Connections " + ioEx.getMessage() + "\n");
        }
    }

}
