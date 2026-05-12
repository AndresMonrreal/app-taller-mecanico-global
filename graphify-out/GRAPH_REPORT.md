# Graph Report - Ulloa  (2026-05-11)

## Corpus Check
- 120 files · ~25,324 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 648 nodes · 885 edges · 62 communities (56 shown, 6 thin omitted)
- Extraction: 98% EXTRACTED · 2% INFERRED · 0% AMBIGUOUS · INFERRED: 17 edges (avg confidence: 0.62)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Community 0|Community 0]]
- [[_COMMUNITY_Community 1|Community 1]]
- [[_COMMUNITY_Community 2|Community 2]]
- [[_COMMUNITY_Community 3|Community 3]]
- [[_COMMUNITY_Community 4|Community 4]]
- [[_COMMUNITY_Community 5|Community 5]]
- [[_COMMUNITY_Community 6|Community 6]]
- [[_COMMUNITY_Community 7|Community 7]]
- [[_COMMUNITY_Community 8|Community 8]]
- [[_COMMUNITY_Community 9|Community 9]]
- [[_COMMUNITY_Community 10|Community 10]]
- [[_COMMUNITY_Community 11|Community 11]]
- [[_COMMUNITY_Community 12|Community 12]]
- [[_COMMUNITY_Community 13|Community 13]]
- [[_COMMUNITY_Community 14|Community 14]]
- [[_COMMUNITY_Community 15|Community 15]]
- [[_COMMUNITY_Community 16|Community 16]]
- [[_COMMUNITY_Community 17|Community 17]]
- [[_COMMUNITY_Community 18|Community 18]]
- [[_COMMUNITY_Community 19|Community 19]]
- [[_COMMUNITY_Community 20|Community 20]]
- [[_COMMUNITY_Community 21|Community 21]]
- [[_COMMUNITY_Community 22|Community 22]]
- [[_COMMUNITY_Community 23|Community 23]]
- [[_COMMUNITY_Community 24|Community 24]]
- [[_COMMUNITY_Community 25|Community 25]]
- [[_COMMUNITY_Community 26|Community 26]]
- [[_COMMUNITY_Community 28|Community 28]]
- [[_COMMUNITY_Community 29|Community 29]]
- [[_COMMUNITY_Community 30|Community 30]]
- [[_COMMUNITY_Community 31|Community 31]]
- [[_COMMUNITY_Community 32|Community 32]]
- [[_COMMUNITY_Community 33|Community 33]]
- [[_COMMUNITY_Community 34|Community 34]]
- [[_COMMUNITY_Community 37|Community 37]]
- [[_COMMUNITY_Community 39|Community 39]]
- [[_COMMUNITY_Community 40|Community 40]]
- [[_COMMUNITY_Community 41|Community 41]]
- [[_COMMUNITY_Community 42|Community 42]]
- [[_COMMUNITY_Community 43|Community 43]]
- [[_COMMUNITY_Community 44|Community 44]]
- [[_COMMUNITY_Community 45|Community 45]]
- [[_COMMUNITY_Community 47|Community 47]]
- [[_COMMUNITY_Community 48|Community 48]]
- [[_COMMUNITY_Community 53|Community 53]]

## God Nodes (most connected - your core abstractions)
1. `cn()` - 72 edges
2. `opencode.md — Frontend (auto-hub-pro)` - 17 edges
3. `opencode.md — Backend (taller-mecanico-api)` - 14 edges
4. `CRUDBase` - 12 edges
5. `Button` - 10 edges
6. `CRUDOrden` - 9 edges
7. `Layout()` - 8 edges
8. `Input` - 7 edges
9. `Label` - 7 edges
10. `Column` - 6 edges

## Surprising Connections (you probably didn't know these)
- `AlertDialogHeader()` --calls--> `cn()`  [EXTRACTED]
  auto-hub-pro/src/components/ui/alert-dialog.tsx → auto-hub-pro/src/lib/utils.ts
- `AlertDialogFooter()` --calls--> `cn()`  [EXTRACTED]
  auto-hub-pro/src/components/ui/alert-dialog.tsx → auto-hub-pro/src/lib/utils.ts
- `BreadcrumbSeparator()` --calls--> `cn()`  [EXTRACTED]
  auto-hub-pro/src/components/ui/breadcrumb.tsx → auto-hub-pro/src/lib/utils.ts
- `BreadcrumbEllipsis()` --calls--> `cn()`  [EXTRACTED]
  auto-hub-pro/src/components/ui/breadcrumb.tsx → auto-hub-pro/src/lib/utils.ts
- `CommandShortcut()` --calls--> `cn()`  [EXTRACTED]
  auto-hub-pro/src/components/ui/command.tsx → auto-hub-pro/src/lib/utils.ts

## Communities (62 total, 6 thin omitted)

### Community 0 - "Community 0"
Cohesion: 0.05
Nodes (38): useIsMobile(), Separator, SheetContent, SheetContentProps, SheetDescription, SheetFooter(), SheetHeader(), SheetOverlay (+30 more)

### Community 1 - "Community 1"
Cohesion: 0.06
Nodes (12): Base, CRUDBase, CRUCDClient, CRUDOrden, CRUDService, CRUDOrden, Client, Order (+4 more)

### Community 2 - "Community 2"
Cohesion: 0.08
Nodes (30): BaseModel, EstimarRequest, ClientBase, ClientCreate, ClientOut, ClientUpdate, Config, Config (+22 more)

### Community 3 - "Community 3"
Cohesion: 0.06
Nodes (34): Autenticación y permisos, Base de datos, code:block1 (taller-mecanico-api/), code:python (# app/services/client_service.py), code:env (# .env.example), code:block12 (test_create_client_success), code:block13 (feat(clients): agregar endpoint de creación), code:python (class ClientBase(BaseModel): ...       # campos comunes) (+26 more)

### Community 4 - "Community 4"
Cohesion: 0.06
Nodes (32): Autenticación, Axios: instancia y configuración, code:block1 (auto-hub-pro/), code:tsx (import { Button } from '@/components/ui/button'), code:tsx (import { cn } from '@/lib/utils'), code:env (# .env.example), code:block13 (feat(clients): agregar tabla de clientes con paginación), code:tsx (const ClientCard = ({ client }: { client: Client }) => {) (+24 more)

### Community 5 - "Community 5"
Cohesion: 0.09
Nodes (19): Route, Route, Route, getRouter(), ClientesRoute, ConfiguracionRoute, FileRoutesByFullPath, FileRoutesById (+11 more)

### Community 6 - "Community 6"
Cohesion: 0.12
Nodes (7): ClienteCreate, Orden, OrdenCreate, Servicio, ServicioCreate, ServicioOrden, VehiculoCreate

### Community 7 - "Community 7"
Cohesion: 0.19
Nodes (16): cn(), Button, ButtonProps, buttonVariants, Calendar(), CalendarDayButton(), Pagination(), PaginationContent (+8 more)

### Community 8 - "Community 8"
Cohesion: 0.1
Nodes (11): Checkbox, HoverCardContent, PopoverContent, Progress, ScrollArea, ScrollBar, Slider, Switch (+3 more)

### Community 9 - "Community 9"
Cohesion: 0.12
Nodes (14): Command, CommandEmpty, CommandGroup, CommandInput, CommandItem, CommandList, CommandSeparator, CommandShortcut() (+6 more)

### Community 10 - "Community 10"
Cohesion: 0.12
Nodes (11): Menubar, MenubarCheckboxItem, MenubarContent, MenubarItem, MenubarLabel, MenubarRadioItem, MenubarSeparator, MenubarShortcut() (+3 more)

### Community 11 - "Community 11"
Cohesion: 0.23
Nodes (10): consumeLastCapturedError(), renderErrorPage(), brandedErrorResponse(), fetch(), getServerEntry(), isCatastrophicSsrErrorBody(), normalizeCatastrophicSsrResponse(), ServerEntry (+2 more)

### Community 12 - "Community 12"
Cohesion: 0.14
Nodes (12): Carousel, CarouselApi, CarouselContent, CarouselContext, CarouselContextProps, CarouselItem, CarouselNext, CarouselOptions (+4 more)

### Community 13 - "Community 13"
Cohesion: 0.15
Nodes (12): labels, OrderStatus, StatusBadge(), styles, Cliente, clientes, Orden, ordenes (+4 more)

### Community 14 - "Community 14"
Cohesion: 0.14
Nodes (12): Feature development workflow, Frontend visual validation (Playwright MCP), Git structure, How the packages relate, Key Tools, MCP Tools: code-review-graph, Monorepo structure, Oracle persistence error guard (+4 more)

### Community 15 - "Community 15"
Cohesion: 0.18
Nodes (8): get_current_user(), create_access_token(), decode_token(), hash_password(), verify_password(), User, login(), register()

### Community 16 - "Community 16"
Cohesion: 0.24
Nodes (7): createVehiculo(), deleteVehiculo(), getVehiculos(), updateVehiculo(), EMPTY_FORM, VehiculoForm, Vehiculo

### Community 17 - "Community 17"
Cohesion: 0.24
Nodes (7): Route, EMPTY_FORM, Route, ServicioForm, Input, Label, labelVariants

### Community 18 - "Community 18"
Cohesion: 0.2
Nodes (8): Modal(), Props, Orden, Route, Servicio, statuses, Vehiculo, Textarea

### Community 19 - "Community 19"
Cohesion: 0.17
Nodes (9): FormControl, FormDescription, FormFieldContext, FormFieldContextValue, FormItem, FormItemContext, FormItemContextValue, FormLabel (+1 more)

### Community 20 - "Community 20"
Cohesion: 0.27
Nodes (7): createCliente(), deleteCliente(), getClientes(), updateCliente(), ClienteForm, EMPTY_FORM, Cliente

### Community 21 - "Community 21"
Cohesion: 0.18
Nodes (7): ChartConfig, ChartContainer, ChartContext, ChartContextProps, ChartLegendContent, ChartTooltipContent, THEMES

### Community 22 - "Community 22"
Cohesion: 0.27
Nodes (5): Layout(), items, Sidebar(), Topbar(), Route

### Community 23 - "Community 23"
Cohesion: 0.2
Nodes (9): DropdownMenuCheckboxItem, DropdownMenuContent, DropdownMenuItem, DropdownMenuLabel, DropdownMenuRadioItem, DropdownMenuSeparator, DropdownMenuShortcut(), DropdownMenuSubContent (+1 more)

### Community 24 - "Community 24"
Cohesion: 0.2
Nodes (9): ContextMenuCheckboxItem, ContextMenuContent, ContextMenuItem, ContextMenuLabel, ContextMenuRadioItem, ContextMenuSeparator, ContextMenuShortcut(), ContextMenuSubContent (+1 more)

### Community 25 - "Community 25"
Cohesion: 0.22
Nodes (8): AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter(), AlertDialogHeader(), AlertDialogOverlay, AlertDialogTitle

### Community 26 - "Community 26"
Cohesion: 0.22
Nodes (8): Table, TableBody, TableCaption, TableCell, TableFooter, TableHead, TableHeader, TableRow

### Community 28 - "Community 28"
Cohesion: 0.32
Nodes (5): Column, DataTable(), Props, Orden, Route

### Community 29 - "Community 29"
Cohesion: 0.25
Nodes (6): DrawerContent, DrawerDescription, DrawerFooter(), DrawerHeader(), DrawerOverlay, DrawerTitle

### Community 30 - "Community 30"
Cohesion: 0.25
Nodes (7): Breadcrumb, BreadcrumbEllipsis(), BreadcrumbItem, BreadcrumbLink, BreadcrumbList, BreadcrumbPage, BreadcrumbSeparator()

### Community 31 - "Community 31"
Cohesion: 0.25
Nodes (7): NavigationMenu, NavigationMenuContent, NavigationMenuIndicator, NavigationMenuList, NavigationMenuTrigger, navigationMenuTriggerStyle, NavigationMenuViewport

### Community 32 - "Community 32"
Cohesion: 0.25
Nodes (7): SelectContent, SelectItem, SelectLabel, SelectScrollDownButton, SelectScrollUpButton, SelectSeparator, SelectTrigger

### Community 33 - "Community 33"
Cohesion: 0.29
Nodes (6): Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle

### Community 34 - "Community 34"
Cohesion: 0.33
Nodes (5): ToggleGroup, ToggleGroupContext, ToggleGroupItem, Toggle, toggleVariants

### Community 40 - "Community 40"
Cohesion: 0.4
Nodes (4): Alert, AlertDescription, AlertTitle, alertVariants

### Community 41 - "Community 41"
Cohesion: 0.4
Nodes (4): InputOTP, InputOTPGroup, InputOTPSeparator, InputOTPSlot

### Community 42 - "Community 42"
Cohesion: 0.67
Nodes (3): Badge(), BadgeProps, badgeVariants

### Community 43 - "Community 43"
Cohesion: 0.5
Nodes (3): Avatar, AvatarFallback, AvatarImage

### Community 44 - "Community 44"
Cohesion: 0.5
Nodes (3): AccordionContent, AccordionItem, AccordionTrigger

## Knowledge Gaps
- **273 isolated node(s):** `VehiculosRoute`, `ServiciosRoute`, `ReportesRoute`, `OrdenesRoute`, `ConfiguracionRoute` (+268 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **6 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `cn()` connect `Community 7` to `Community 0`, `Community 8`, `Community 9`, `Community 10`, `Community 12`, `Community 13`, `Community 17`, `Community 18`, `Community 19`, `Community 21`, `Community 22`, `Community 23`, `Community 24`, `Community 25`, `Community 26`, `Community 29`, `Community 30`, `Community 31`, `Community 32`, `Community 33`, `Community 34`, `Community 40`, `Community 41`, `Community 42`, `Community 43`, `Community 44`, `Community 47`?**
  _High betweenness centrality (0.169) - this node is a cross-community bridge._
- **Why does `Button` connect `Community 7` to `Community 0`, `Community 12`, `Community 16`, `Community 17`, `Community 18`, `Community 20`, `Community 28`?**
  _High betweenness centrality (0.011) - this node is a cross-community bridge._
- **Are the 4 inferred relationships involving `CRUDBase` (e.g. with `CRUCDClient` and `CRUDOrden`) actually correct?**
  _`CRUDBase` has 4 INFERRED edges - model-reasoned connections that need verification._
- **What connects `VehiculosRoute`, `ServiciosRoute`, `ReportesRoute` to the rest of the system?**
  _273 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `Community 0` be split into smaller, more focused modules?**
  _Cohesion score 0.05 - nodes in this community are weakly interconnected._
- **Should `Community 1` be split into smaller, more focused modules?**
  _Cohesion score 0.06 - nodes in this community are weakly interconnected._
- **Should `Community 2` be split into smaller, more focused modules?**
  _Cohesion score 0.08 - nodes in this community are weakly interconnected._