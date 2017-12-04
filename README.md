##  Monex Ultra-C and Kiosoft Software Scripted Transaction

### Make a payment transaction in minutes with autocom and ultra-c.kiooft scripts.

```
/**
    TO USE autocom and or autocom.exe for payment systems with ultra-C  and any other
    payment peripherials you would need a custom license from the publisher of this software.
*/

dofile("include.sss");


/*
    This script:
        - connect to ultra-C device on serial. See include.sss for comport setting.
        - disables power saving.
        - prints device information, Can be commented
        - sends a transaction of 5c to kiosk host server. Make sure you've ran sync.sss, and host ip's are setup properly.
        - waits for completion. The function  saleTransacrionReq in include.sss, prints all transaction id's
        - for success / failure, can open another device as
            - ssh
            - serial
            - a socket
            This example togles DTR and RTS, wires from the serial, I have 2 LED'd in contral parralel on these.
            and creates a file in local folder success, or failed.
            - Can log transaction in log files, and recicle them (disabled in autcom code , (work in proigress for Windows))
*/


function main(zzz)
{
    Debug(0);  // debugs Device level received send buffers
    print("Starting \n");

    local serial = Device(S.SERIAL_PORT, "serial", BINARY);

    print("\n\nDISABLE POWER---------------------------- \n");
    enquire(serial, T.Disable_Power_Request, T.Disable_Power_Response, null);
    Sleep(1000);

    //
    // get device info
    //
    // print("\n\nGET DEVICE INFO---------------------------------\n");

    local devinfo = enquire(serial, T.Get_Device_Information_Request, T.Get_Device_Information_Response,null);
    printDevInfo(devinfo);

    //
    // sale transaction second parameter is in cents
    // the function saleTransacrionReq() is located in include.sss script
    //
    print("\n\nSALE TRANSACTION 5 cents  2 minutes----------------------------\n");
    if(saleTransacrionReq(serial, 5, 120000))
    {
        print ("\n DTR 0 RTS 1  Successful Transaction\n");

        //
        // can notify the serial back, some peripherial can rely on DTR/RTS which is not used by the ultra-C
        //
        serial.escape("DTR", 0);
        serial.escape("RTS", 1);

        // or create a file so a third party software can trigger some events
        local file = Device("file://success.txt", "file", TEXT);
        file.putsnl("success");
        Sleep(3000);
        file.remove();
    }
    else
    {
        print ("\n DTR 0 RTS 1  Failed Transaction\n");
        serial.escape("DTR", 1);
        serial.escape("RTS", 0);
        local file = Device("file://fail.txt", "file", TEXT);
        file.putsnl("failed");
        Sleep(3000);
        file.remove();
    }

    print ("\n\n ENABLE POWER SAVE ----------------------------------\n");
    enquire(serial, T.Sw_Enable_Power_Request, T.Sw_Enable_Power_Response,null);
	Sleep(3000);
    return Exit();
}

```




