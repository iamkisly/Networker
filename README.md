# Networker
A simple to use TCP and UDP networking library for .NET Core and .NET Framework.

## Features (v1)
* TCP
* UDP
* Low memory footprint
* Handle thousands of simultaneous connections
* Incredibly fast serialization using ZeroFormatter
* Plug in your choice of logging

## Planned Features (v2)
* Plug in your choice of IOC
* Encryption

## Supported Frameworks
* .NET Standard 1.6
* .NET Core
* .NET Framework 4.5.2

## Installing
Install-Package Networker

## Getting Started

Networker uses a client-server architecture for communication.

Many clients can connect to a single server.

### Creating a TCP Server
```csharp
public class Program
    {
        static void Main(string[] args)
        {
            var server = new NetworkerServerBuilder()
            .UseConsoleLogger()
            .UseTcp(1050)
            .RegisterPacketHandler<ChatMessagePacket, ChatMessagePacketHandler>()
            .Build<DefaultServer>()
            .Start();
        }
    }
```

### Creating a TCP Client
```csharp
public class Program
    {
        static void Main(string[] args)
        {
            var client = new NetworkerClientBuilder()
            .UseConsoleLogger()
            .UseIp("127.0.0.1")
            .UseTcp(1050)
            .RegisterPacketHandler<ChatMessagePacket, ChatMessageReceivedPacketHandler>()
            .Build<DefaultClient>()
            .Connect();
        }
    }
```

### Creating a Packet
```csharp
public class ChatMessagePacket : NetworkerPacketBase
    {
        [Index(0)]
        public virtual string Message { get; set; }

        [Index(1)]
        public virtual string Sender { get; set; }
    }
```

### Creating a Server Packet Handler
```csharp
public class ChatMessagePacketHandler : ServerPacketHandlerBase<ChatMessagePacket>
    {
        private readonly ITcpConnectionsProvider connectionsProvider;

        public ChatMessagePacketHandler(ITcpConnectionsProvider connectionsProvider)
        {
            this.connectionsProvider = connectionsProvider;
        }

        public override void Handle(INetworkerConnection sender, ChatMessagePacket packet)
        {
            foreach(var tcpConnection in this.connectionsProvider.Provide())
            {
                tcpConnection.Send(packet);
            }
        }
    }
```


### Creating a Client Packet Handler
```csharp
public class ChatMessageReceivedPacketHandler : PacketHandlerBase<ChatMessagePacket>
    {
        public override void Handle(ChatMessagePacket packet)
        {
            var window = MainWindow.Instance;

            window.Dispatcher.Invoke(() =>
                                     {
                                         window.MessageListBox.Items.Add($"{packet.Sender}: {packet.Message}");
                                     });
        }
    }
```



