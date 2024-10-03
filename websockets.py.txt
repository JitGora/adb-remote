import asyncio
import websockets
import subprocess

async def handle_connection(websocket, path):
    async for message in websocket:
        key_event = message
        print(f"Received key event: {key_event}")

        # Execute the ADB command
        process = await asyncio.create_subprocess_exec(
            'adb', 'shell', 'input', 'keyevent', key_event,
            stdout=asyncio.subprocess.PIPE,
            stderr=asyncio.subprocess.PIPE
        )

        stdout, stderr = await process.communicate()

        if stdout:
            print(f'[stdout] {stdout.decode()}')
        if stderr:
            print(f'[stderr] {stderr.decode()}')

        # Optionally send a confirmation message back to the client
        await websocket.send(f"ADB command {key_event} executed")

async def main():
    async with websockets.serve(handle_connection, "0.0.0.0", 8765):
        await asyncio.Future()  # Run forever

if __name__ == "__main__":
    asyncio.run(main())
