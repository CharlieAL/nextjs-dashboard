# App Router

El app router en next js viene incluido por defecto, no es necesario instalarlo, este router es muy parecido al de react-router-dom, pero con algunas diferencias.

Solo que cada carpeta que se cree dentro de la carpeta app y contenga un archivo page.tsx, sera una ruta de la app, por ejemplo:



- app
  - page.tsx -->  http://localhost:3000/
  - dashboard
    - page.tsx -->  http://localhost:3000/dashboard
    - customers
      - page.tsx -->  http://localhost:3000/dashboard/customers

# Images

El < Image /> Componente de next js nos permite optimizar las imagenes de nuestra app, para que estas carguen mas rapido, y tambien nos permite hacer lazy loading de las imagenes, es decir que las imagenes se carguen solo cuando el usuario las necesite, esto es muy bueno para el performance de la app.

```jsx
import Image from 'next/image' 
```

Este es un ejemplo cuando un usuario esta en un dispositivo movil no carga la imagen '/hero-desktop.png' porque no se ve en el viewport entonces no la renderiza ni la carga, y el otro cuando esta en un dispositivo de escritorio igual con la imagen '/hero-mobile.png'.


```jsx
<Image 
  src={'/hero-desktop.png'} 
  alt='logo image' 
  width={1000} 
  height={760} 
  className='hidden md:block' 
/>
```
```jsx
<Image 
  src={'/hero-mobile.png'} 
  alt='logo image' 
  width={560} 
  height={620} 
  className='block md:hidden' 
/>
```
# Link 

el link en next js es un componente que nos permite navegar entre paginas de nuestra aplicacion

```jsx 
import Link from 'next/link'
```
 Una app con next js hace code-splitting no es como en react con la SPA que se carga todo el codigo de la app en el navegador, en next js se carga solo el codigo necesario para la pagina que se esta visitando, esto es muy bueno para el performance de la app.

 code-splitting es una tecnica que nos permite dividir el codigo de nuestra app en pequeños pedazos de codigo, y solo cargar el codigo que se necesita para la pagina que se esta visitando, esto es muy bueno para el performance de la app.

```jsx
<Link href="/about">
    <a>About</a>
</Link>
```
Si tienes varios links en tu app, y quieres que diferenciar cual es el link que esta activo, puedes usar el hook 'usePathname' para obtener el pathname y comopararlo con los links que tengas ejemplo.


```jsx
import { usePathname } from 'next/navigation';
import clsx from 'clsx'; // libreria para concatenar clases de css

// Map of links to display in the side navigation.
// Depending on the size of the application, this would be stored in a database.
const links = [
  { name: 'Home', href: '/dashboard', icon: HomeIcon },
  {
    name: 'Invoices',
    href: '/dashboard/invoices',
    icon: DocumentDuplicateIcon,
  },
  { name: 'Customers', href: '/dashboard/customers', icon: UserGroupIcon },
];

{links.map((link) => {
  const LinkIcon = link.icon;
    return (
      <Link
        key={link.name}
        href={link.href}
        className={
          clsx('
            flex h-[48px]hover:text-blue-600 
            md:flex-none md:justify-start',
            {
              'bg-sky-100 text-blue-600': 
                pathname === link.href,
            },
        )}
      >
        <p className="hidden md:block">{link.name}</p>
      </Link>
    );
})}
```

# Fetching Data

En next js se puede usar el useEffect pero el problema es que se ejecuta en el lado del cliente se estaria perdiendo tiempo de carga. 
En nextjs por defecto los componenetes se estan renderizando en el servidor **(react server component)**, esto quiere decir que pueden ser asincronos y poner un await con el fetch a una api externa o a una api interna.

**El SDK Postgres de Vercel proporciona protección contra inyecciones SQL**
```jsx
import { sql } from '@vercel/postgres';

export default async function Page() {
  const rows = await sql`slect * from users`
  console.log(rows) --> [{...}, {...}, {...}]

  return (
    <p>Dashboard page</p>
  )
}
```

esto puede tener un detalle que si son varias consultas que tarden 1seg o 3 seg por cada consulta, hay 2 formas de solucionarlo. 
1. archivo loading.tsx / loading.js y  automaticamente se va a renderizar ese componente mientras se esta haciendo la consulta a la api.
2. < Suspense /> Componente de react pasarle un fallback que es un componente que se va a renderizar mientras se esta haciendo la consulta a la api o a la base de datos.


##  Loading 
el problema es que todo este componente se tiene que esperer a que se haga la consulta 
 
- app
  - ui
    - RevenueChart.tsx
    - LatestInvoices.tsx
  - dashboard
    - page.tsx
    - loading.tsx

archivo page.tsx
```jsx 
export default async function Page() {
  const revenue = await fetchRevenue(); ---> // tarda 3 seg
  const latestInvoices = await fetchLatestInvoices();

  return (
    <div>
      <RevenueChart revenue={revenue}  />
      <LatestInvoices latestInvoices={latestInvoices} />
    </div>
  )
}
```

archivo loading.tsx
```jsx
import DashboardSkeleton from "../ui/skeletons";

export default function Loading(){
  return (
    <DashboardSkeleton/>
  )
}
```


## Suspense




tenemos que envolver el componente con el componente < Suspense /> y pasarle un fellback que es un componente que se va a renderizar mientras se esta haciendo la consulta a la api o a la base de datos.

Esto seria en el componente padre
```jsx
export default async function Page() {
  const latestInvoices = await fetchLatestInvoices();

  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <RevenueChart />
      </Suspense>
      <LatestInvoices latestInvoices={latestInvoices} />
    </div>
  )
}
```

Esto seria en el componente hijo RevenueChart.tsx
```jsx

export default async function RevenueChart() {
  const revenue = await fetchRevenue();
  {...}
}
```
ahora el componente RevenueChart.tsx va a esperar a que se haga la consulta a la api, y cuando se haga la consulta a la api se va a renderizar el componente RevenueChart.tsx en el navegador. solo esa pequeña parte va a tardar 3 seg, pero el resto de la app se va a renderizar en el servidor y se va a enviar al navegador, esto es muy bueno para el performance de la app.

### ⬆ es Streaming
![App Screenshot](https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fserver-rendering-with-streaming.png&w=3840&q=75&dpl=dpl_AGVpExNSxGb3dC5jrZYnL2rzPEsj)

# Que son request waterfalls?

Una "waterfalls" se refiere a una secuencia de peticiones de red que dependen de la finalización de peticiones anteriores. En el caso de la obtención de datos, cada solicitud sólo puede comenzar una vez que la solicitud anterior ha devuelto los datos.

![App Screenshot](https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fsequential-parallel-data-fetching.png&w=3840&q=75&dpl=dpl_AGVpExNSxGb3dC5jrZYnL2rzPEsj)

```jsx
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices(); // espera por fetchRevenue() para empezar
```

hay ocaciones donde es necesario el waterfalls, si una petición depende de la otra.

# Estado en el URL

En next js te facilita mucho usar el navegador como estado por medio de params.

esto se utiliza en el componente input
```jsx  
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams, usePathname, useRouter } from 'next/navigation';
 
export default function Search() {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();
 
  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
  }
}
// ...
```

```jsx
<input
  placeholder={placeholder}
  onChange={(e) => { handleSearch(e.target.value)}}
  defaultValue={searchParams.get('query')?.toString()}
/>
```
handleSearch va en el onChange del input, y cuando el usuario escriba algo en el input va a cambiar el estado del url, y cuando el usuario haga enter en el teclado va a enviar el estado del url a la api.

esto se utiliza en el componente page.tsx
```jsx
export default async function Page({
  searchParams,
}: {
  searchParams?: {
    query?: string;
    page?: string;
  };
}) {
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;
// ...
```
en next el archivo page.tsx recive por default los params de la url.

---
Esto sirve para que cada vez que se cambie el query o el currentPage se ponga el skeleton.

el suspense solo funciona una vez, pero al agregarle el key se vuelve a ejecutar el suspense.
```jsx
<Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
  <Table query={query} currentPage={currentPage} />
</Suspense>
```


# Server Actions 

Permite ejecutar codigo asincrono en el servidor, y no en el cliente.
Te quita la necesidad de crear una api cada vez que quieras mutar datos.

ejemplo
```jsx
export default function Page() {
  // Action
  async function create(formData: FormData) {
    'use server';
 
    // Logica para mutar data...
  }
 
  // Invocar la acción mediante el atributo "action"
  return <form action={create}>...</form>;
}
```
---
## use server

```jsx
'use server'
```
Se puede user el 'use server'. para crear las funciones que se van a ejecutar en el servidor.

Si se coloca al inicio de un archivo marca que todas las funciones que se exportan en este archivo son de
servidor y por lo tanto no se ejecuta ni se envían al cliente.

Ejemplo de como se usa un server action con un formulario.

Archivo action.tsx
```tsx
'use server'

export async function createInvoice(fromData:FormData) {
  console.log('createInvoice', fromData) --> FormData { ... }
}
```

Archivo componente.tsx
```tsx
import { createInvoice } from '@/lib/actions';

export default function Form() {
  return (
    <form action={createInvoice}>
        {...}
    </form>
  )
}

```

##  Revalidate & redirect

Next.js dispone de una caché "Client-side Router" que almacena los segmentos de ruta en el navegador del usuario durante un tiempo. Junto con la precarga, esta caché garantiza que los usuarios puedan navegar rápidamente entre las rutas al tiempo que reduce el número de peticiones realizadas al servidor.

Dado que estás actualizando los datos mostrados en la ruta de facturas, quieres borrar esta caché y lanzar una nueva petición al servidor. Puedes hacerlo con la función revalidatePath de Next.js:

```tsx
'use server'
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createInvoice(formData:FormData) {
  // Logica para mutar data...
  revalidatePath('/dashboard/invoices'); ---> // revalida la ruta
  redirect('/dashboard/invoices'); ---> // redirecciona a la ruta
}
``` 

















