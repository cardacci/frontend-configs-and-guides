#### Custom types with interfaces

interface <span style="color: #4EC990;">Address</span> {
	line1: <span style="color: #4EC990;">string</span>;
	line2: <span style="color: #4EC990;">string</span>;
	postalCode: <span style="color: #4EC990;">string</span>;
	province: <span style="color: #4EC990;">string</span>;
	region?: <span style="color: #4EC990;">string</span>;
}

interface <span style="color: #4EC990;">Contact</span> extends <span style="color: #4EC990;">Address</span> {
	birthdate?: <span style="color: #4EC990;">Date</span>;
	id: <span style="color: #4EC990;">number</span>;
	name: <span style="color: #4EC990;">string</span>;
}

let primaryContact: <span style="color: #4EC990;">Contact</span> = {
	birthdate: new Date('21-04-1989'),
	id: 12345,
	line1: 'Libres del Sur',
	line2: '135',
	name: 'Gabriel Fernandez',
	postalCode: '7109',
	province: 'Buenos Aires'
}
