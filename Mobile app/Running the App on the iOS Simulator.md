## Step 1: Install Xcode

1. Open the **App Store** on your Mac, search **Xcode**, and install it.
   
https://github.com/user-attachments/assets/959a4ce0-3469-4e14-8553-c823e9282b19


2. Open Xcode once and let it finish installing its components.

<img width="1900" height="1063" alt="Screenshot 2026-09-28 at 13 16 50" src="https://github.com/user-attachments/assets/2b3c3dda-f442-4821-b9a0-d102bdb99464" />


<img width="1905" height="1078" alt="Screenshot 2026-09-28 at 13 17 29" src="https://github.com/user-attachments/assets/95687a35-f520-4dff-9442-d5f039c43c01" />





3. Launch the Simulator:
   * Press `⌘ + Space`, type **DeviceHub**, and hit Enter (Xcode 27+).
   * Under **Simulators**, click an iPhone to boot it.
   * On Xcode 26 or earlier: **Xcode > Open Developer Tool > Simulator**.
  
<img width="1884" height="1033" alt="Screenshot 2026-09-28 at 13 19 22" src="https://github.com/user-attachments/assets/b1153c34-2e04-49e1-abf3-ca3efa7458b3" />


<img width="1602" height="978" alt="Screenshot 2026-09-28 at 13 19 34" src="https://github.com/user-attachments/assets/2ca423a8-645f-464e-a768-649639b68919" />



## Step 2: Run the App on the Simulator

From the project root (`SamuraiTax-Mobile`):

```bash
npm install
npx expo run:ios
```

The first build takes a few minutes. The app opens on the simulator automatically.
