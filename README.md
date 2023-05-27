# MiniUSB-Host-Shield

Nicolás Amaro Muñoz Read
Instituto de Innovación y Tecnologia Aplicada

RESUMEN:

Esta es una documentación técnica sobre el funcionamiento del HW-244 MINI USB Host Shield y como hacer su conexión a una Arduino Uno.

El HW-244 MINI USB Host Shield es una placa usb con un micro MAX 3421E, Cuenta con una interfaz de comunicación SPI y permite la conexión y control de una amplia variedad de dispositivos USB. La ventaja del HW-244 es que convierte el dispositivo que esté conectado a el Mini USB en Maestro (Host) y cualquier otro dispositivo o periferico que se conecte al usb se convierte en esclavo. 

Para programar en el IDE de Arduino es necesario descargar y ag

CODIGO 1: 

    //PRUEBA DE CONECCIONES PARA COMPROBAR QUE ESTÉ TODO CORRECTO EN LA COMUNICACIÓN ENTRE EL HW-244 Y LA ARDUINO UNO
    //FUNCIONA PARA CUALQUIER OTRA PLACA DE ARDUINO
 
	include <usbhid.h>
	#include <hiduniversal.h>
	#include <hidescriptorparser.h>
	#include <usbhub.h>
	#include "pgmstrings.h"

	// Satisfy the IDE, which needs to see the include statment in the ino too.
	#ifdef dobogusinclude
	#include <spi4teensy3.h>
	#endif
	#include <SPI.h>

	class HIDUniversal2 : public HIDUniversal
	{
	public:
	    HIDUniversal2(USB *usb) : HIDUniversal(usb) {};

	protected:
	    uint8_t OnInitSuccessful();
	};

	uint8_t HIDUniversal2::OnInitSuccessful()
	{
	    uint8_t    rcode;

	    HexDumper<USBReadParser, uint16_t, uint16_t>    Hex;
	    ReportDescParser                                Rpt;

	    if ((rcode = GetReportDescr(0, &Hex)))
		goto FailGetReportDescr1;

	    if ((rcode = GetReportDescr(0, &Rpt)))
		goto FailGetReportDescr2;

	    return 0;

	FailGetReportDescr1:
	    USBTRACE("GetReportDescr1:");
	    goto Fail;

	FailGetReportDescr2:
	    USBTRACE("GetReportDescr2:");
	    goto Fail;

	Fail:
	    Serial.println(rcode, HEX);
	    Release();
	    return rcode;
	}

	USB Usb;
	//USBHub Hub(&Usb);
	HIDUniversal2 Hid(&Usb);
	UniversalReportParser Uni;

	void setup()
	{
	  Serial.begin( 9600 );
	#if !defined(__MIPSEL__)
	  while (!Serial); // Wait for serial port to connect - used on Leonardo, Teensy and other boards with built-in USB CDC serial connection
	#endif
	  Serial.println("Start");

	  if (Usb.Init() == -1)
	      Serial.println("OSC did not start.");

	  delay( 200 );

	  if (!Hid.SetReportParser(0, &Uni))
	      ErrorMessage<uint8_t>(PSTR("SetReportParser"), 1  );
	}

	void loop()
	{
	    Usb.Task();
	}




CODIGO 2: 

    //CODIGO DE EJEMPLO DE LA LIBRERIA DEL USBHost shield PARA LA CONECCION ENTRE UNA ARDUINO UNO JUNTO CON EL USB HOST SHIELD Y UN MOUSE, SE PUEDE ENCONTRAR EN LOS EJEMPLOS DE LA LIBRERIA DESCARGADA

	#include <hidboot.h>
	#include <usbhub.h>

	// Satisfy the IDE, which needs to see the include statment in the ino too.
	#ifdef dobogusinclude
	#include <spi4teensy3.h>
	#endif
	#include <SPI.h>

	class MouseRptParser : public MouseReportParser
	{
	protected:
		void OnMouseMove	(MOUSEINFO *mi);
		void OnLeftButtonUp	(MOUSEINFO *mi);
		void OnLeftButtonDown	(MOUSEINFO *mi);
		void OnRightButtonUp	(MOUSEINFO *mi);
		void OnRightButtonDown	(MOUSEINFO *mi);
		void OnMiddleButtonUp	(MOUSEINFO *mi);
		void OnMiddleButtonDown	(MOUSEINFO *mi);
	};
	void MouseRptParser::OnMouseMove(MOUSEINFO *mi)
	{
	    Serial.print("dx=");
	    Serial.print(mi->dX, DEC);
	    Serial.print(" dy=");
	    Serial.println(mi->dY, DEC);
	};
	void MouseRptParser::OnLeftButtonUp	(MOUSEINFO *mi)
	{
	    Serial.println("L Butt Up");
	};
	void MouseRptParser::OnLeftButtonDown	(MOUSEINFO *mi)
	{
	    Serial.println("L Butt Dn");
	};
	void MouseRptParser::OnRightButtonUp	(MOUSEINFO *mi)
	{
	    Serial.println("R Butt Up");
	};
	void MouseRptParser::OnRightButtonDown	(MOUSEINFO *mi)
	{
	    Serial.println("R Butt Dn");
	};
	void MouseRptParser::OnMiddleButtonUp	(MOUSEINFO *mi)
	{
	    Serial.println("M Butt Up");
	};
	void MouseRptParser::OnMiddleButtonDown	(MOUSEINFO *mi)
	{
	    Serial.println("M Butt Dn");
	};

	USB     Usb;
	USBHub     Hub(&Usb);
	HIDBoot<USB_HID_PROTOCOL_MOUSE>    HidMouse(&Usb);

	MouseRptParser                               Prs;

	void setup()
	{
	    Serial.begin( 9600 );
	#if !defined(__MIPSEL__)
	    while (!Serial); // Wait for serial port to connect - used on Leonardo, Teensy and other boards with built-in USB CDC serial connection
	#endif
	    Serial.println("Start");

	    if (Usb.Init() == -1)
		Serial.println("OSC did not start.");

	    delay( 200 );

	    HidMouse.SetReportParser(0, &Prs);
	}

	void loop()
	{
	  Usb.Task();
	}
