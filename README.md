using UnityEngine;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading;
public class ARCarWiFiController : MonoBehaviour
{
public float speed = 5f; // car speed
private bool isRunning = false;
private TcpListener tcpListener;
private Thread tcpListenerThread;
void Start()
{
// Start TCP server
tcpListenerThread = new Thread(new ThreadStart(ListenForMessages));
tcpListenerThread.IsBackground = true;
tcpListenerThread.Start();
}
void ListenForMessages()
{
try
{
tcpListener = new TcpListener(IPAddress.Any, 5000);
tcpListener.Start();
Debug.Log("TCP Server started on port 5000");
while (true)
{
using (TcpClient client = tcpListener.AcceptTcpClient())
using (NetworkStream stream = client.GetStream())
{
byte[] buffer = new byte[1024];
int length = stream.Read(buffer, 0, buffer.Length);
if (length > 0)
{
string msg = Encoding.ASCII.GetString(buffer, 0, length).Trim();
Debug.Log("Received: " + msg);
if (msg == "run")
isRunning = true;
else if (msg == "stop")
isRunning = false;
}
}
}
}
catch (SocketException ex)
{
Debug.LogError("SocketException: " + ex.ToString());
}
}
void Update()
{
if (isRunning)
{
transform.Translate(Vector3.forward * speed * Time.deltaTime);
}
}
void OnApplicationQuit()
{
if (tcpListener != null)
tcpListener.Stop();
if (tcpListenerThread != null && tcpListenerThread.IsAlive)
tcpListenerThread.Abort();
}
}
