<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ACUARIO VALPARAISO POS</title>
    <!-- Cargamos React y Babel -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Iconos Lucide -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://unpkg.com/lucide-react@latest"></script>
    
    <style>
        .custom-scrollbar::-webkit-scrollbar { width: 6px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #0ea5e9; border-radius: 20px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #0284c7; }
        #error-display { display: none; position: fixed; top: 0; left: 0; width: 100%; background: red; color: white; padding: 20px; z-index: 9999; font-family: monospace; }
    </style>
</head>
<body class="bg-slate-50">
    <!-- Div para mostrar errores si la app falla -->
    <div id="error-display"></div>
    <div id="root">
        <div class="h-screen flex items-center justify-center text-blue-900 font-bold">
            Cargando Acuario POS...
        </div>
    </div>

    <script type="text/babel">
        // Capturador de errores para que no se quede en blanco
        window.onerror = function(msg, url, lineNo, columnNo, error) {
            const display = document.getElementById('error-display');
            display.style.display = 'block';
            display.innerHTML = 'Error en la App: ' + msg + ' (Línea: ' + lineNo + ')';
            return false;
        };

        const { useState, useEffect, useMemo } = React;
        const { 
            LayoutGrid, History, ShoppingCart, Search, Plus, Trash2, 
            DollarSign, X, CheckCircle2, Clock, Ticket, Gift, 
            Printer, Minus, CreditCard, Banknote, BarChart3, 
            ChevronRight, Utensils, Waves, Anchor, Compass, 
            MessageCircle, Users 
        } = lucideReact;

        const INITIAL_PRODUCTS = [
            { id: 1, name: "Entrada General Adulto", price: 4900, category: "Entradas", color: "bg-sky-50", icon: Ticket },
            { id: 2, name: "Entrada Niño (2-12 años)", price: 3900, category: "Entradas", color: "bg-sky-50", icon: Ticket },
            { id: 3, name: "Entrada Convenio BCI", price: 3675, category: "Entradas", color: "bg-cyan-50", icon: Ticket },
            { id: 5, name: "Peluche Tiburón Blanco", price: 12000, category: "Zoovenir", color: "bg-blue-50", icon: Gift },
            { id: 6, name: "Gorra Acuario Valpo", price: 6500, category: "Zoovenir", color: "bg-blue-50", icon: Gift },
            { id: 11, name: "Combo Fish & Chips", price: 7500, category: "Comida", color: "bg-teal-50", icon: Utensils },
            { id: 13, name: "Bebida Océano 500ml", price: 2000, category: "Comida", color: "bg-teal-50", icon: Utensils },
        ];

        const GUIDES = ["Guía Marina", "Guía Sebastián", "Guía Javiera", "Guía Rodrigo", "Guía Ignacia"];

        const App = () => {
            const [currentView, setCurrentView] = useState('pos');
            const [activeCategory, setActiveCategory] = useState('Todas');
            const [cart, setCart] = useState([]);
            const [salesHistory, setSalesHistory] = useState([]);
            const [searchTerm, setSearchTerm] = useState('');
            const [isCheckoutOpen, setIsCheckoutOpen] = useState(false);
            const [paymentMethod, setPaymentMethod] = useState('efectivo');
            const [cashReceived, setCashReceived] = useState('');
            const [currentTime, setCurrentTime] = useState(new Date());
            const [selectedGuide, setSelectedGuide] = useState('');
            const [tourHistory, setTourHistory] = useState([]);

            useEffect(() => {
                const timer = setInterval(() => setCurrentTime(new Date()), 1000);
                return () => clearInterval(timer);
            }, []);

            const filteredProducts = INITIAL_PRODUCTS.filter(p => {
                const matchesSearch = p.name.toLowerCase().includes(searchTerm.toLowerCase());
                const matchesCategory = activeCategory === 'Todas' || p.category === activeCategory;
                return matchesSearch && matchesCategory;
            });

            const total = cart.reduce((acc, item) => acc + (item.price * item.quantity), 0);

            const productSalesReport = useMemo(() => {
                const report = {};
                INITIAL_PRODUCTS.forEach(p => {
                    report[p.id] = { ...p, soldCount: 0, totalRevenue: 0 };
                });
                salesHistory.forEach(sale => {
                    sale.items.forEach(item => {
                        if (report[item.id]) {
                            report[item.id].soldCount += item.quantity;
                            report[item.id].totalRevenue += (item.price * item.quantity);
                        }
                    });
                });
                return Object.values(report).sort((a, b) => b.soldCount - a.soldCount);
            }, [salesHistory]);

            const addToCart = (product) => {
                setCart(prev => {
                    const existing = prev.find(item => item.id === product.id);
                    if (existing) {
                        return prev.map(item => item.id === product.id ? { ...item, quantity: item.quantity + 1 } : item);
                    }
                    return [...prev, { ...product, quantity: 1 }];
                });
            };

            const updateQuantity = (id, delta) => {
                setCart(prev => prev.map(item => {
                    if (item.id === id) {
                        const newQty = item.quantity + delta;
                        return newQty > 0 ? { ...item, quantity: newQty } : item;
                    }
                    return item;
                }).filter(item => item.quantity > 0));
            };

            const removeFromCart = (id) => {
                setCart(prev => prev.filter(item => item.id !== id));
            };

            const finalizeSale = () => {
                const newSale = {
                    id: `AV-${Date.now().toString().slice(-6)}`,
                    time: new Date().toLocaleTimeString(),
                    items: [...cart],
                    total: total,
                    method: paymentMethod,
                };
                setSalesHistory([newSale, ...salesHistory]);
                setCart([]);
                setCashReceived('');
                setIsCheckoutOpen(false);
            };

            const handleCreateTour = () => {
                if (!selectedGuide) return;
                const now = new Date();
                const minutes = now.getMinutes();
                const roundedMinutes = Math.round(minutes / 5) * 5;
                const baseDate = new Date(now);
                baseDate.setMinutes(roundedMinutes);
                baseDate.setSeconds(0);
                const tourDate = new Date(baseDate.getTime() + 15 * 60000);
                const tourTime = tourDate.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });

                const newTour = {
                    id: Date.now(),
                    guide: selectedGuide,
                    time: tourTime,
                    created: now.toLocaleTimeString()
                };
                setTourHistory([newTour, ...tourHistory]);
                const message = `🌊 *ACUARIO VALPARAISO POS*: Nuevo recorrido.\n\n👤 *Guía*: ${selectedGuide}\n⏰ *Hora*: ${tourTime}`;
                window.open(`https://wa.me/?text=${encodeURIComponent(message)}`, '_blank');
            };

            const formatMoney = (val) => {
                return new Intl.NumberFormat('es-CL', { style: 'currency', currency: 'CLP', minimumFractionDigits: 0 }).format(val);
            };

            return (
                <div className="flex h-screen overflow-hidden font-sans">
                    {/* Sidebar */}
                    <aside className="w-24 lg:w-64 bg-[#001d3d] text-white flex flex-col shrink-0 border-r border-blue-900 shadow-xl">
                        <div className="p-6 mb-8 flex items-center gap-3">
                            <div className="bg-sky-500 p-2 rounded-xl text-white shadow-lg"><Waves size={24} /></div>
                            <div className="hidden lg:block">
                                <h1 className="font-black text-sm uppercase leading-none">Acuario</h1>
                                <span className="text-sky-400 font-bold text-[10px] tracking-widest">VALPARAÍSO POS</span>
                            </div>
                        </div>
                        <nav className="flex-1 px-4 space-y-2">
                            <button onClick={() => setCurrentView('pos')} className={`w-full flex items-center gap-4 px-4 py-4 rounded-2xl transition-all ${currentView === 'pos' ? 'bg-sky-600' : 'text-slate-400 hover:bg-slate-800'}`}>
                                <ShoppingCart size={22} /><span className="font-bold text-sm hidden lg:block">Vender</span>
                            </button>
                            <button onClick={() => setCurrentView('tours')} className={`w-full flex items-center gap-4 px-4 py-4 rounded-2xl transition-all ${currentView === 'tours' ? 'bg-sky-600' : 'text-slate-400 hover:bg-slate-800'}`}>
                                <Compass size={22} /><span className="font-bold text-sm hidden lg:block">Recorridos</span>
                            </button>
                            <button onClick={() => setCurrentView('history')} className={`w-full flex items-center gap-4 px-4 py-4 rounded-2xl transition-all ${currentView === 'history' ? 'bg-sky-600' : 'text-slate-400 hover:bg-slate-800'}`}>
                                <History size={22} /><span className="font-bold text-sm hidden lg:block">Historial</span>
                            </button>
                            <button onClick={() => setCurrentView('reports')} className={`w-full flex items-center gap-4 px-4 py-4 rounded-2xl transition-all ${currentView === 'reports' ? 'bg-sky-600' : 'text-slate-400 hover:bg-slate-800'}`}>
                                <BarChart3 size={22} /><span className="font-bold text-sm hidden lg:block">Reportes</span>
                            </button>
                        </nav>
                        <div className="p-6 mt-auto">
                            <div className="bg-blue-900/30 p-4 rounded-2xl flex items-center gap-3 border border-blue-800/50">
                                <Anchor size={18} className="text-sky-400" />
                                <span className="text-[10px] font-black uppercase text-sky-200 hidden lg:block">Muelle Prat</span>
                            </div>
                        </div>
                    </aside>

                    {/* Main Content */}
                    <main className="flex-1 flex flex-col overflow-hidden bg-slate-50">
                        <header className="h-20 bg-white border-b flex items-center justify-between px-8 shrink-0">
                            <div className="flex items-center gap-2 bg-sky-50 px-4 py-2 rounded-full text-sky-600 border border-sky-100">
                                <Clock size={16} /><span className="text-xs font-bold font-mono uppercase tracking-tighter">{currentTime.toLocaleTimeString()}</span>
                            </div>
                            <h2 className="text-[10px] font-black text-slate-400 uppercase tracking-widest hidden md:block italic">Sistema POS • Acuario Valparaíso</h2>
                        </header>

                        <div className="flex-1 overflow-hidden">
                            {currentView === 'pos' && (
                                <div className="flex h-full">
                                    <div className="flex-1 p-6 overflow-y-auto custom-scrollbar">
                                        <div className="flex flex-col md:flex-row justify-between mb-8 gap-4">
                                            <div className="relative w-full md:w-80">
                                                <Search className="absolute left-4 top-1/2 -translate-y-1/2 text-slate-400" size={18} />
                                                <input type="text" placeholder="Buscar..." className="w-full pl-12 pr-6 py-3 bg-white border rounded-2xl text-sm outline-none shadow-sm" value={searchTerm} onChange={(e) => setSearchTerm(e.target.value)} />
                                            </div>
                                            <div className="flex gap-2 bg-white p-1 rounded-2xl border shadow-sm overflow-x-auto">
                                                {['Todas', 'Entradas', 'Zoovenir', 'Comida'].map(cat => (
                                                    <button key={cat} onClick={() => setActiveCategory(cat)} className={`px-4 py-2 rounded-xl text-[10px] font-black uppercase transition-all whitespace-nowrap ${activeCategory === cat ? 'bg-sky-600 text-white' : 'text-slate-400 hover:bg-slate-50'}`}>{cat}</button>
                                                ))}
                                            </div>
                                        </div>
                                        <div className="grid grid-cols-2 lg:grid-cols-4 gap-6">
                                            {filteredProducts.map(product => (
                                                <button key={product.id} onClick={() => addToCart(product)} className={`${product.color} p-6 rounded-[2.5rem] border-2 border-transparent hover:border-sky-300 transition-all flex flex-col justify-between h-48 text-left shadow-sm hover:shadow-xl active:scale-95`}>
                                                    <div className="bg-white/80 p-2 rounded-xl text-sky-700 w-fit shadow-sm"><product.icon size={20} /></div>
                                                    <div>
                                                        <p className="text-[9px] font-black uppercase text-sky-400 mb-1 leading-none">{product.category}</p>
                                                        <h3 className="font-extrabold text-slate-800 text-sm mb-1 leading-tight">{product.name}</h3>
                                                        <p className="text-sky-700 font-black text-xl">{formatMoney(product.price)}</p>
                                                    </div>
                                                </button>
                                            ))}
                                        </div>
                                    </div>
                                    <div className="w-full md:w-[420px] bg-white border-l flex flex-col shadow-2xl">
                                        <div className="p-6 border-b flex justify-between items-center bg-sky-50/30">
                                            <h2 className="font-black text-xl flex items-center gap-2 text-sky-900 uppercase tracking-tighter"><ShoppingCart size={20} /> Pedido</h2>
                                            <button onClick={() => setCart([])} className="text-[10px] font-black text-red-400 uppercase tracking-widest hover:text-red-600">Vaciar</button>
                                        </div>
                                        <div className="flex-1 overflow-y-auto p-4 space-y-3 custom-scrollbar bg-slate-50/30">
                                            {cart.map(item => (
                                                <div key={item.id} className="bg-white p-4 rounded-2xl border flex items-center gap-3 shadow-sm hover:border-sky-100 transition-all">
                                                    <div className="flex-1">
                                                        <p className="text-sm font-black text-slate-800 leading-tight mb-1">{item.name}</p>
                                                        <p className="text-[10px] text-sky-400 font-bold uppercase">{formatMoney(item.price)}</p>
                                                    </div>
                                                    <div className="flex items-center bg-slate-50 rounded-xl p-1 border">
                                                        <button onClick={() => updateQuantity(item.id, -1)} className="w-7 h-7 flex items-center justify-center hover:bg-white rounded-lg text-slate-400"><Minus size={14}/></button>
                                                        <span className="w-8 text-center text-xs font-black">{item.quantity}</span>
                                                        <button onClick={() => updateQuantity(item.id, 1)} className="w-7 h-7 flex items-center justify-center hover:bg-white rounded-lg text-slate-400"><Plus size={14}/></button>
                                                    </div>
                                                    <div className="text-right flex flex-col items-end">
                                                        <p className="font-black text-sky-700 text-sm">{formatMoney(item.price * item.quantity)}</p>
                                                        <button onClick={() => removeFromCart(item.id)} className="text-red-300 hover:text-red-500 p-1 transition-colors"><Trash2 size={14} /></button>
                                                    </div>
                                                </div>
                                            ))}
                                            {cart.length === 0 && (
                                                <div className="h-full flex flex-col items-center justify-center text-slate-300 opacity-50 py-20">
                                                    <ShoppingCart size={48} className="mb-4" />
                                                    <p className="font-black text-[10px] uppercase tracking-widest">Carrito Vacío</p>
                                                </div>
                                            )}
                                        </div>
                                        <div className="p-8 bg-[#001d3d] text-white rounded-t-[3rem] shadow-inner">
                                            <div className="flex justify-between items-center mb-6 px-2">
                                                <span className="text-xs font-black text-sky-400 uppercase tracking-widest">Total Neto</span>
                                                <span className="text-4xl font-black text-white tracking-tighter">{formatMoney(total)}</span>
                                            </div>
                                            <button onClick={() => setIsCheckoutOpen(true)} disabled={cart.length === 0} className="w-full bg-sky-500 text-white py-5 rounded-[2rem] font-black text-xl uppercase tracking-widest shadow-xl shadow-sky-500/20 active:scale-95 transition-all disabled:opacity-50 disabled:bg-slate-800">Cobrar</button>
                                        </div>
                                    </div>
                                </div>
                            )}

                            {currentView === 'tours' && (
                                <div className="flex-1 p-10 overflow-y-auto bg-sky-50/30">
                                    <div className="max-w-5xl mx-auto">
                                        <h2 className="text-4xl font-black text-slate-900 tracking-tighter uppercase mb-10">Recorridos Guiados</h2>
                                        <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
                                            <div className="bg-white p-8 rounded-[3rem] border shadow-xl">
                                                <label className="block text-[10px] font-black text-slate-400 uppercase mb-4 tracking-widest text-center">Seleccionar Guía</label>
                                                <div className="space-y-2 mb-8">
                                                    {GUIDES.map(guide => (
                                                        <button key={guide} onClick={() => setSelectedGuide(guide)} className={`w-full p-4 rounded-2xl text-left font-bold border-2 transition-all ${selectedGuide === guide ? 'bg-sky-50 border-sky-500 text-sky-700 shadow-md' : 'bg-slate-50 border-transparent text-slate-500 hover:bg-slate-100'}`}>{guide}</button>
                                                    ))}
                                                </div>
                                                <button onClick={handleCreateTour} disabled={!selectedGuide} className="w-full bg-emerald-600 text-white py-5 rounded-[2rem] font-black flex items-center justify-center gap-3 active:scale-95 transition-all shadow-lg shadow-emerald-100">
                                                    <MessageCircle size={22} /> Notificar por WSP
                                                </button>
                                            </div>
                                            <div className="lg:col-span-2 space-y-4">
                                                <h3 className="font-black uppercase text-xs tracking-widest text-slate-400 ml-4 mb-4">Próximos (Redondeo 5 min)</h3>
                                                {tourHistory.map(tour => (
                                                    <div key={tour.id} className="bg-white p-6 rounded-[2.5rem] border shadow-sm flex items-center justify-between group hover:shadow-lg transition-all">
                                                        <div className="flex items-center gap-6">
                                                            <div className="bg-sky-50 text-sky-600 p-4 rounded-3xl font-black text-xl shadow-inner">{tour.time}</div>
                                                            <div><p className="font-black text-slate-800 text-lg leading-none mb-1 uppercase tracking-tight">{tour.guide}</p><p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest italic leading-none">Inicia en 15m aprox.</p></div>
                                                        </div>
                                                    </div>
                                                ))}
                                                {tourHistory.length === 0 && <div className="p-20 text-center text-slate-300 font-bold uppercase text-xs border-2 border-dashed rounded-[3rem]">Sin recorridos hoy</div>}
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            )}

                            {currentView === 'history' && (
                                <div className="flex-1 p-10 overflow-y-auto bg-slate-100/30">
                                    <div className="max-w-4xl mx-auto space-y-4">
                                        <h2 className="text-4xl font-black text-slate-900 tracking-tighter uppercase mb-8">Bitácora de Ventas</h2>
                                        {salesHistory.map(sale => (
                                            <div key={sale.id} className="bg-white p-8 rounded-[2.5rem] border shadow-sm flex items-center justify-between hover:border-sky-100 transition-all">
                                                <div className="flex items-center gap-6">
                                                    <div className="w-14 h-14 bg-sky-100 text-sky-600 rounded-2xl flex items-center justify-center font-black text-xs">POS</div>
                                                    <div><p className="text-sm font-black text-slate-800 leading-none mb-1">{sale.id}</p><p className="text-[10px] text-slate-400 font-bold uppercase">{sale.time} • {sale.method.toUpperCase()}</p></div>
                                                </div>
                                                <p className="text-2xl font-black text-slate-900 tracking-tight">{formatMoney(sale.total)}</p>
                                            </div>
                                        ))}
                                        {salesHistory.length === 0 && <p className="text-center py-20 text-slate-400 font-bold italic uppercase tracking-widest">Sin ventas registradas</p>}
                                    </div>
                                </div>
                            )}

                            {currentView === 'reports' && (
                                <div className="flex-1 p-10 overflow-y-auto">
                                    <div className="max-w-5xl mx-auto">
                                        <h2 className="text-4xl font-black text-slate-900 tracking-tighter uppercase mb-10">Reporte de Inventario</h2>
                                        <div className="bg-white rounded-[3rem] border border-slate-200 shadow-sm overflow-hidden">
                                            <table className="w-full text-left">
                                                <thead className="bg-slate-50 border-b">
                                                    <tr>
                                                        <th className="px-8 py-5 text-[10px] font-black text-slate-400 uppercase tracking-widest">Producto Marino</th>
                                                        <th className="px-8 py-5 text-[10px] font-black text-slate-400 uppercase tracking-widest text-center">Unidades</th>
                                                        <th className="px-8 py-5 text-[10px] font-black text-slate-

