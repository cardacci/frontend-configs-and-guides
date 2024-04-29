#### Custom types with interfaces

```typescript
type ContactName = string;

enum ContactStatus {
	Active = "active",
	Inactive = "inactive",
	New = "new"
}

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
	name: ContactName;
	status: ContactStatus;
}

let primaryContact: Contact = {
	birthdate: new Date('21-04-1989'),
	id: 12345,
	line1: 'Libres del Sur',
	line2: '135',
	name: 'Gabriel Fernandez',
	postalCode: '7109',
	province: 'Buenos Aires',
	status: ContactStatus.Active
}
```
