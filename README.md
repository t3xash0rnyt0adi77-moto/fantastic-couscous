# fantastic-couscous
Explore the full Bitcoin ecosystem with mempool.space, or be your own explorer and self-host your own instance with one-click installation on popular Raspberry Pi fullnode distros including Umbrel, Raspiblitz, Start9, and more!
# python3
from binascii import unhexlify

raw_tx = "020000000001014d5c5951b57d3ffcf35f8219a01c3686647acf732f0a92d731b7e77ee78f359a3201000000ffffffff010000000000000000326a304920616d204d656c76696e2043617276616c686f20612070726f66696369656e74206d617468656d6174696369616e2e01403c04daa97bb2d711843af7c22eefea1ca301f5cf984306a4cef6a005848c06653beaa531183efb1401ba2dcf24e82ba140725185cec78a34e9487e073ec5317400000000"
tx = unhexlify(raw_tx)

# find 0x6a bytes (OP_RETURN)
for i in range(len(tx)):
    if tx[i] == 0x6a:  # OP_RETURN
        j = i + 1
        if j >= len(tx):
            continue
        # handle pushdata opcodes
        if tx[j] == 0x4c:          # OP_PUSHDATA1
            if j+1 >= len(tx):
                continue
            length = tx[j+1]
            start = j+2
        elif tx[j] == 0x4d:        # OP_PUSHDATA2
            if j+2 >= len(tx):
                continue
            length = tx[j+1] | (tx[j+2] << 8)
            start = j+3
        elif tx[j] == 0x4e:        # OP_PUSHDATA4
            if j+4 >= len(tx):
                continue
            length = tx[j+1] | (tx[j+2] << 8) | (tx[j+3] << 16) | (tx[j+4] << 24)
            start = j+5
        else:
            # single-byte push (0x01..0x4b)
            length = tx[j]
            start = j+1

        payload = tx[start:start+length]
        print("OP_RETURN at byte", i, "payload length", length)
        print("hex:", payload.hex())
        try:
            print("utf8:", payload.decode('utf-8'))
        except UnicodeDecodeError:
            print("utf8: <not valid UTF-8>")
        # stop after the first OP_RETURN found
        break