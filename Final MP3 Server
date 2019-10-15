import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.SocketException;
import java.util.ArrayList;

/**
 * A MP3 Server for sending mp3 files over a socket connection.
 *
 * @author Daniel Hong, 7
 * @author Gabe Efsits, 7
 * @version 2019-04-12
 */

public class MP3Server {

    private final ArrayList<ClientHandler> clients;
    private final int port;

    private MP3Server(int port) {
        this.clients = new ArrayList<ClientHandler>();
        this.port = port;
    }

    private void start() {
        try {
            ServerSocket serverSocket = new ServerSocket(this.port);
            while (true) {
                Socket socket = serverSocket.accept();
                try {
                    Runnable r = new ClientHandler(socket);
                    Thread t = new Thread(r);
                    clients.add((ClientHandler) r);
                    t.start();
                    System.out.println("New Client Connected");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {

        MP3Server server = new MP3Server(8080);
        server.start();
    }
}
/**
 * Class - ClientHandler
 * <p>
 * This class implements Runnable, and will contain the logic for handling responses and requests to
 * and from a given client. The threads you create in MP3Server will be constructed using instances
 * of this class.
 *
 * @author Daniel Hong, 7
 * @author Gabe Efsits, 7
 * @version 2019-04-12
 */
final class ClientHandler implements Runnable {

    private ObjectInputStream ois;
    private ObjectOutputStream oos;

    public ClientHandler(Socket clientSocket) {
        if (clientSocket == null) {
            throw new IllegalArgumentException();
        }
        try {
            ois = new ObjectInputStream(clientSocket.getInputStream());
            oos = new ObjectOutputStream(clientSocket.getOutputStream());
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    /**
     * This method is the start of execution for the thread. See the handout for more details on what
     * to do here.
     */
    public void run() {
        //Implement run method. Remember to listen for the client's input indefinitely
        SongRequest received = null;

        while (true) {
            try {
                received = (SongRequest) ois.readObject();

                if (received.isDownloadRequest()) {
                    String fileName = received.getArtistName() + " - " + received.getSongName() + ".mp3";

                    //Make sure file in record returns true when correct name and song is sent
                    if (!fileInRecord(fileName)) {
                        getOutputStream().writeObject(new SongHeaderMessage(true,
                                "N/A", "N/A", -1));
                    } else if (fileInRecord(fileName)) {
                        byte[] songData = readSongData(fileName);
                        if (songData != null && songData.length > 0) {
                            getOutputStream().writeObject(new SongHeaderMessage(true,
                                    received.getSongName(), received.getArtistName(), songData.length));
                            //Check to make sure client is receiving data
                            sendByteArray(songData);
                            System.out.println("Song data sent for " + fileName);

                        }
                    }
                } else {
                    getOutputStream().writeObject(new SongHeaderMessage(false, "LIST",
                            "LIST", -1));
                    sendRecordData();
                }
            } catch (IOException | ClassNotFoundException e) {
                if (e instanceof SocketException) {
                    System.out.println("Client Disconnected");
                } else {
                    e.printStackTrace();
                }
                break;
            }
        }
        //Close the streams
        try {
            getInputStream().close();
            getOutputStream().close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * Searches the record file for the given filename.
     *
     * @param fileName the fileName to search for in the record file
     * @return true if the fileName is present in the record file, false if the fileName is not
     */
    private static boolean fileInRecord(String fileName) {
        //check
        //Implement fileInRecord
        //String filePath = "D:\\IntelliJ\\Project 4\\s19-project-4\\record.txt";
        String filePath = "./record.txt";
        boolean fileInRecord = false;
        try {
            BufferedReader br = new BufferedReader(new FileReader(filePath));
            String line;
            while ((line = br.readLine()) != null) {
                if (line.equals(fileName)) {
                    fileInRecord = true;
                }
            }
            br.close();

        } catch (Exception e) {
            return false;
        }
        return fileInRecord;
    }

    /**
     * Read the bytes of a file with the given name into a byte array.
     *
     * @param fileName the name of the file to read
     * @return the byte array containing all bytes of the file, or null if an error occurred
     */
    private static byte[] readSongData(String fileName) {
        //File file = new File("D:\\IntelliJ\\Project 4\\s19-project-4\\songDatabase\\" + fileName);
        File file = new File("./songDatabase/" + fileName);
        byte[] songData = new byte[(int) file.length()];

        try {
            FileInputStream fis = new FileInputStream(file);
            fis.read(songData);
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
        return songData;
    }

    /**
     * Split the given byte array into smaller arrays of size 1000, and send the smaller arrays
     * to the client using SongDataMessages.
     *
     * @param songData the byte array to send to the client
     */
    private void sendByteArray(byte[] songData) throws IOException {
        int totalSent = 0;
        int leftOver = 0;
        for (int i = 0; i < (songData.length / 1000); i++) {
            byte[] songDataPart = new byte[1000];
            System.arraycopy(songData, totalSent, songDataPart, 0, 1000);
            getOutputStream().writeObject(new SongDataMessage(songDataPart));
            totalSent += 1000;
            //System.out.println("sent data chunk total=>" + totalSent);
            getOutputStream().flush();
        }
        leftOver = songData.length - totalSent;
        if (leftOver > 0) {
            byte[] songDataPart = new byte[leftOver];

            System.arraycopy(songData, totalSent, songDataPart, 0, leftOver);
            getOutputStream().writeObject(new SongDataMessage(songDataPart));
            //System.out.println("sent data chunk total=>" + totalSent);
            getOutputStream().flush();
        }
        getOutputStream().close();
    }

    /**
     * Read ''record.txt'' line by line again, this time formatting each line in a readable
     * format, and sending it to the client. Send a ''null'' value to the client when done, to
     * signal to the client that you've finished sending the record data.
     */
    private void sendRecordData() {
        //Implement sendRecordData
        String filePath = "./record.txt";

        try {
            BufferedReader br = new BufferedReader(new FileReader(filePath));
            String line;
            while ((line = br.readLine()) != null) {
                String songName = line.substring(0, line.indexOf('-') - 1);
                String artistName = line.substring(line.indexOf('-') + 2, line.length() - 4);
                getOutputStream().writeObject("\"" + songName + "\" by: " + artistName);
                getOutputStream().flush();
            }

            getOutputStream().close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    public ObjectInputStream getInputStream() {
        return ois;
    }

    public ObjectOutputStream getOutputStream() {
        return oos;
    }

}
