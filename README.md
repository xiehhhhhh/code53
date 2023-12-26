# code53
import java.io.DataInputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;


public class Reciever {

    public static void main(String[] args){
        try{
            ServerSocket serverSocket = new ServerSocket(8080);
            String savePath = "I:\\";
            Socket socket = null;

            while(true){
                socket = serverSocket.accept();
                ServerTheard thread = new ServerTheard(socket,savePath);
                thread.start();

                InetAddress address = socket.getInetAddress();
                System.out.println("Client IP:"+ address.getHostAddress());
            }
        }catch(Exception e){
            throw new RuntimeException(e);
        }
        
    }
}

class ServerTheard extends Thread{
    private Socket socket = null;
    private String savePath = null;

    public ServerTheard(Socket socket,String savePath){
        this.socket = socket;
        this.savePath = savePath;
    }

    @Override
    public void run(){
        try{
            DataInputStream dis = new DataInputStream(socket.getInputStream());
            String fileName = dis.readUTF();
            long fileLength = dis.readLong();

            File directory = new File(savePath+ socket.getInetAddress());
            if(!directory.exists()){
                directory.mkdirs();
            }
            File file = new File(directory.getAbsolutePath() + File.separatorChar + fileName);
            int loop = 1;
            while(file.exists()){
                file = new File(directory.getAbsolutePath() + File.separatorChar +"("+ loop +")"+ fileName);
                loop += 1;
            }
            FileOutputStream fos = new FileOutputStream(file);

            byte[] bytes = new byte[1024];
            int length = 0;
            while((length=dis.read(bytes,0,bytes.length))!=-1){
                fos.write(bytes);
                fos.flush();
            }
            dis.close();
            fos.close();
            socket.close();
        }catch(Exception e){
            System.out.println("ERROR");
        }
    }
}
