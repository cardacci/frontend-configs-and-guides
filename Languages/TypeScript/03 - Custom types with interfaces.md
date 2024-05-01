#### Custom types with interfaces

We will see several examples of how to use types and interfaces in TypeScript.

```typescript
type ContactBirthdate = Date | number | string;
type ContactName = string;

enum ContactStatus {
	Active = 'active',
	Inactive = 'inactive',
	New = 'new'
}
/*
An alternative to creating an enum could be the following, although using an enum reduces errors, especially when it's exported and needs to be used from another file. Additionally, it's less maintainable because if a value changes, it needs to be updated in every implementation. In contrast, using an enum only requires changing it in the declaration.
*/
type ContactStatus = 'active', 'inactive', 'new'

interface Address {
	line1: string;
	line2: string;
	postalCode: string;
	province: string;
	region?: string;
}

interface Contact extends Address {
	birthdate?: ContactBirthdate;
	id: number;
	name: ContactName;
	status: ContactStatus;
}

// Here are two different ways to make use of two interfaces:
interface AddressableContact extends Address, Contact {}
type AddressableContact = Address & Contact

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

function getBirthDate(contact: Contact) {
	if (typeof contact.birthDate === 'number') {
		return new Date(contact.birthDate);
	} else if (typeof contact.birthDate === 'string') {
		return Date.parse(contact.birthDate)
	} else {
		// In this flow, TypeScript will understand that birthdate is of type Date.
		return contact.birthDate
	}
}
```
