# Mock-React-Socket-App
a friend of a friend needed help with a react websockets app and i wrote a mock app trying to understand what he needed, thought i should upload the mock app here to flex my react knowledge 😂😂

```tsx
// jotai is a global state management solution, uses "atoms" that you can subscribe to to fetch data across any component
// when data is updated using the atom setter function, it updates the state of any component using it
const socketAtom = useAtom<SocketClient | undefined>(undefined);

function SocketApp() {
  const [, setSocketClient] = useAtom(socketAtom);

  // when the app starts, connect to the socket, save the returned socketClient so we can use it to handle message events
  useEffect(() => {
    const socketClient = socket.connect("ws://...");

    setSocketClient(socketClient);

    return () => socketClient.disconnect();
  }, [setSocketClient]);

  // have a separate data visualizer component because we want to save the data from the socket to state
  // reminder that state updates will re-render the component
  // we disconnect the client when the current component lifecycle ends, so it wouldn't be ideal to handle message events here
  return (
    <DataVisualizer />
  )
}

interface Data {
  id: string;
}

function DataVisualizer() {
  // subscribe to the socket client atom
  const [socketClient] = useAtom(socketAtom);
  const [data, setData] = useState<Data[]>([]);

  useEffect(() => {
    // handle events in this component because if we re-render this, it will not close our existing connection
    const messageEvent = socketClient.on("message", (message) => {
      setData(JSON.parse(message)); // im assuming the data is just sent as json
    });

    // when this component re-renders, simply close the old event and a new one will be established in the next lifecycle
    return () => {
      messageEvent.close();
    }
  }, [socketClient, data]); // update this component when "data" changes

  // this function will request new data from the socket, use it where-ever you want to
  const requestNewData = () => {
    socketClient.send(JSON.stringify(data.map(datum => datum.id)));
  };

  // render data to screen
  return (
    <Data>
      {data.map((datum, key) => (
        <div key={key}>
          {datum.dataThatNeedsToBeVisualized}
        </div>
      )}
    </Data>
  )
}
```
