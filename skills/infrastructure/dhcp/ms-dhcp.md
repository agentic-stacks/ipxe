# Microsoft DHCP Server Configuration

Configure via **Start → Administrative Tools → DHCP** (the DHCP MMC snap-in).

## Basic Boot Configuration

1. Right-click **Server Options** → **Configure Options**
2. Set **067 Bootfile Name** to `http://192.168.1.20/boot.ipxe`

For SAN boot, set **017 Root Path** instead.

## Chainloading with User-Class Policies

### Step 1: Define the iPXE User Class

1. Right-click your DHCP server → **Define User Classes**
2. Click **Add**
3. Set:
   - Display name: `iPXE`
   - Description: `iPXE clients`
   - ID: `iPXE` (type in the ASCII column)
4. Click **OK**

### Step 2: Set Default Boot File

1. Go to **Server Options** → **Configure Options**
2. Set **067 Bootfile Name** to `undionly.kpxe`

This sends the chainload binary to all non-iPXE (legacy PXE) clients.

### Step 3: Create a Policy for iPXE Clients

**Windows Server 2012 and later:**

1. Right-click **Policies** → **New Policy**
2. Name: `iPXE Boot`
3. Add condition: **User Class** equals **iPXE**
4. In the policy options, set **067 Bootfile Name** to `http://192.168.1.20/boot.ipxe`

**Windows Server 2008 R2:**

1. Go to **Server Options** → **Configure Options**
2. Click the **Advanced** tab
3. Select **iPXE** from the **User class** dropdown
4. Set **067 Bootfile Name** to `http://192.168.1.20/boot.ipxe`

## Result

- Legacy PXE clients receive `undionly.kpxe` (chainloads iPXE)
- iPXE clients receive `http://192.168.1.20/boot.ipxe` (the real boot script)
- No infinite boot loop
