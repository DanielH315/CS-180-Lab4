import org.omg.CosNaming.NamingContextPackage.NotFound;

import java.io.*;
import java.net.Socket;

/**
 * An MP3 Client to request .mp3 files from a server and receive them over the socket connection.
 *
 * @author Daniel Hong, 7
 * @author Gabe Efsits, 7
 * @version 2019-04-12
 */
public class MP3Client {

    private final String server;
    private final int port;

    private MP3Client() {
        //server = "66.70.189.118";
        //port = 9478;
        server = "localhost";
        port = 8080;
    }

    public static void main(String[] args) {
        MP3Client client = new MP3Client();
        //create loop to handle multiple requests
        client.start();
        //Implement main
    }

    private void start() {
        ObjectOutputStream oos;
        Socket socket;
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String menu;
        String songName;
        String artistName;
        boolean validMenu = false;

        try {
            socket = new Socket(server, port);
            oos = new ObjectOutputStream(socket.getOutputStream());
        } catch (IOException e) {
            e.printStackTrace();
            return;
        }

        Runnable r = new ResponseListener(socket);
        Thread t = new Thread(r);
        t.start();
        boolean clientExit = false;

        try {
            while (!clientExit) {
                // take user input to see what request to make
                do {
                    System.out.println("list / download/ exit: ");
                    menu = br.readLine();
                    if ("list".equals(menu) || "download".equals(menu)) {
                        validMenu = true;
                    }
                } while (!validMenu);
                if ("exit".equals(menu)) {
                    clientExit = true;
                } else if ("download".equals(menu)) {
                    System.out.println("Enter song name: ");
                    songName = br.readLine();

                    System.out.println("Enter artist name: ");
                    artistName = br.readLine();

                    //we need the input of the user (hardcoded Ringtones for now)
                    //SongRequest sr = new SongRequest(true, "Ringtones", "Marimba");
                    SongRequest sr = new SongRequest(true, songName, artistName);
                    oos.writeObject(sr);
                    System.out.println("Request sent!");
                    synchronized (socket) {
                        try {
                            socket.wait();
                        } catch (InterruptedException ie) {
                            ie.printStackTrace();
                        }
                    }
                } else {
                    SongRequest sr = new SongRequest(false);
                    oos.writeObject(sr);
                    System.out.println("Request sent!");
                    synchronized (socket) {
                        try {
                            socket.wait();
                        } catch (InterruptedException ie) {
                            ie.printStackTrace();
                        }
                    }
                }
            }
            return;
        } catch (IOException e) {
            e.printStackTrace();
            return;
        }
    }
}

// Create song request and send to server

/**
 * This class implements Runnable, and will contain the logic for listening for
 * server responses. The threads you create in MP3Server will be constructed using
 * instances of this class.
 *
 * @author Daniel Hong, 7
 * @author Gabe Efsits, 7
 * @version 2019-04-12
 */
final class ResponseListener implements Runnable {

    private ObjectInputStream ois;
    private final Socket clientSocket;

    public ResponseListener(Socket clientSocket) {

        if (clientSocket == null) {
            throw new IllegalArgumentException();
        }
        this.clientSocket = clientSocket;
        try {
            ois = new ObjectInputStream(clientSocket.getInputStream());
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    /**
     * Listens for a response from the server.
     * <p>
     * Continuously tries to read a SongHeaderMessage. Gets the artist name, song name, and file size from that header,
     * and if the file size is not -1, that means the file exists. If the file does exist, the method then subsequently
     * waits for a series of SongDataMessages, takes the byte data from those data messages and writes it into a
     * properly named file.
     */
    public void run() {
        //Implement run
        SongHeaderMessage received = null;
        Object object;
        boolean requestProcessed = false;
        try {
            object = ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
            return;
        }
        do {
            //Check to make sure this is how to read in iO
            if (object instanceof SongHeaderMessage) {
                received = (SongHeaderMessage) object;
                if (received.isSongHeader()) {
                    int fileSize = received.getFileSize();
                    int readTotal = 0;
                    SongDataMessage dataMsg;
                    byte[] songData = new byte[fileSize];
                    System.out.println("Song: " + received.getSongName() + ", Size: " + fileSize);

                    //Check to make sure this is the correct way to read in the file
                    for (int i = 0; i < fileSize / 1000; i++) {
                        try {
                            dataMsg = (SongDataMessage) ois.readObject();
                            byte[] data = dataMsg.getData();
                            System.arraycopy(data, 0, songData, readTotal, 1000);
                            readTotal += 1000;
                        } catch (IOException | ClassNotFoundException e) {
                            e.printStackTrace();
                        }
                    }
                    //read the remaining
                    int remain = fileSize - readTotal;
                    if (remain > 0) {
                        try {
                            dataMsg = (SongDataMessage) ois.readObject();
                            byte[] data = dataMsg.getData();
                            System.arraycopy(data, 0, songData, readTotal, remain);
                        } catch (IOException | ClassNotFoundException e) {
                            e.printStackTrace();
                        }
                    }
                    //Now, we got all data in songData


                    this.writeByteArrayToFile(songData, received.getSongName() + ".mp3");
                    requestProcessed = true;
                    synchronized (clientSocket) {
                        clientSocket.notifyAll();
                    }
                } else if (!received.isSongHeader()) {  //Song list

                    try {
                        while ((object = ois.readObject()) != null) {
                            System.out.println((String) object);
                        }
                    } catch (IOException | ClassNotFoundException e) {
                        if (e instanceof EOFException) {
                            System.out.println("");
                        } else {
                            e.printStackTrace();
                        }
                    }
                    requestProcessed = true;
                    synchronized (clientSocket) {
                        clientSocket.notifyAll();
                    }
                }
            }
        } while (received != null && !requestProcessed);
    }


    /**
     * Writes the given array of bytes to a file whose name is given by the fileName argument.
     *
     * @param songBytes the byte array to be written
     * @param fileName  the name of the file to which the bytes will be written
     */
    private void writeByteArrayToFile(byte[] songBytes, String fileName) {
        //File file = new File("D:\\IntelliJ\\Project 4\\s19-project-4\\songDatabase\\" + fileName);
        File file = new File("./" + fileName);
        FileOutputStream fos = null;

        try {
            fos = new FileOutputStream(file);
            fos.write(songBytes);
        } catch (IOException ioe) {
            ioe.printStackTrace();
        } finally {
            try {
                if (fos != null) {
                    fos.close();
                }
            } catch (IOException ioe) {
                ioe.printStackTrace();
            }
        }
    }

    public ObjectInputStream getOis() {
        return ois;
    }
}
