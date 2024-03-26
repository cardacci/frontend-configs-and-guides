#### Custom types with interfaces

```typescript
interface Address {
	line1: string;
	line2: string;
	postalCode: string;
	province: string;
	region?: string;
}

interface Contact extends Address {
	birthdate?: Date;
	id: number;
	name: string;
}

let primaryContact: Contact = {
	birthdate: new Date('21-04-1989'),
	id: 12345,
	line1: 'Libres del Sur',
	line2: '135',
	name: 'Gabriel Fernandez',
	postalCode: '7109',
	province: 'Buenos Aires'
}
```
