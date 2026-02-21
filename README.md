<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ACUARIO VALPARAISO POS</title>
    <!-- Librerías vía CDN -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://unpkg.com/lucide-react@latest"></script>
    
    <style>
        .custom-scrollbar::-webkit-scrollbar { width: 6px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #0ea5e9; border-radius: 20px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #0284c7; }
    </style>
</head>
<body class="bg-slate-50">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo } = React;
        const { 
            LayoutGrid, History, ShoppingCart, Search, Plus, Trash2, 
            DollarSign, X, CheckCircle2, Clock, Ticket, Gift, 
            Printer, Minus, CreditCard, Banknote, BarChart3, 
            ChevronRight, Utensils, Waves, Anchor, Compass, 
            MessageCircle, Users 
        } = lucideReact;

        // CONFIGURACIÓN DE PRODUCTOS
        const INITIAL_PRODUCTS = [
            { id: 1, name: "Entrada Adulto", price: 4900, category: "Entradas", color: "bg-sky-50", icon: Ticket },
            { id: 2, name: "Entrada Niño", price: 3900, category: "Entradas", color: "bg-sky-50", icon: Ticket },
            { id: 3, name: "Entrada BCI", price: 3675, category: "Entradas", color: "bg-cyan-50", icon: Ticket },
            { id: 5, name: "Peluche Tiburón", price: 12000, category: "Zoovenir", color: "bg-blue-50", icon: Gift },
            { id: 6, name: "Gorra Acuario", price: 6500, category: "Zoovenir", color: "bg-blue-50", icon: Gift },
            { id: 11, name: "Combo Fish & Chips", price: 7500, category: "Comida", color: "bg-teal-50", icon: Utensils },
            { id: 13, name: "Bebida 500ml", price: 2000, category: "Comida", color: "bg-teal-50", icon: Utensils },
        ];

        const GUIDES = ["Guía Marina", "Guía Sebastián", "Guía Javiera", "Guía Rodrigo"];

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
                setIsCheckoutOpen(false);
                setCashReceived('');
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
                const message = `🌊 *ACUARIO VALPARAISO*: Nuevo recorrido.\n\n👤 *Guía*: ${selectedGuide}\n⏰ *Hora*: ${tourTime}`;
                window.open(`https://wa.me/?text=${encodeURIComponent(message)}`, '_blank');
            };

            const formatMoney = (val) => {
                return new Intl.NumberFormat('es-CL', { style: 'currency', currency: 'CLP', minimumFractionDigits: 0 }).format(val);
            };

            return (
                <div className="flex h-screen overflow-hidden font-sans">
                    {/* Sidebar Náutico */}
                    <aside className="w-20 lg:w-64 bg-[#001d3d] text-white flex flex-col shrink-0 border-r border-blue-900 shadow-xl">
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

                    {/* Main Content Area */}
                    <main className="flex-1 flex flex-col overflow-hidden bg-slate-50">
                        <header className="h-16 bg-white border-b flex items-center justify-between px-8 shrink-0">
                            <div className="flex items-center gap-2 bg-sky-50 px-4 py-1 rounded-full text-sky-600 border border-sky-100">
                                <Clock size={16} /><span className="text-xs font-bold font-mono">{currentTime.toLocaleTimeString()}</span>
                            </div>
                            <h2 className="text-[10px] font-black text-slate-400 uppercase tracking-widest hidden md:block">Terminal POS Acuario</h2>
                        </header>

                        <div className="flex-1 overflow-hidden">
                            {currentView === 'pos' && (
                                <div className="flex h-full">
                                    <div className="flex-1 p-6 overflow-y-auto custom-scrollbar">
                                        <div className="flex flex-col md:flex-row justify-between mb-8 gap-4">
                                            <div className="relative w-full md:w-64">
                                                <Search className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400" size={16} />
                                                <input type="text" placeholder="Buscar..." className="w-full pl-10 pr-4 py-2 bg-white border rounded-xl text-sm outline-none shadow-sm" value={searchTerm} onChange={(e) => setSearchTerm(e.target.value)} />
                                            </div>
                                            <div className="flex gap-1 bg-white p-1 rounded-xl border shadow-sm overflow-x-auto">
                                                {['Todas', 'Entradas', 'Zoovenir', 'Comida'].map(cat => (
                                                    <button key={cat} onClick={() => setActiveCategory(cat)} className={`px-4 py-1.5 rounded-lg text-[9px] font-black uppercase transition-all whitespace-nowrap ${activeCategory === cat ? 'bg-sky-600 text-white' : 'text-slate-400 hover:bg-slate-50'}`}>{cat}</button>
                                                ))}
                                            </div>
                                        </div>
                                        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                                            {filteredProducts.map(product => (
                                                <button key={product.id} onClick={() => addToCart(product)} className={`${product.color} p-5 rounded-[2rem] border-2 border-transparent hover:border-sky-300 transition-all flex flex-col justify-between h-44 text-left shadow-sm hover:shadow-lg active:scale-95`}>
                                                    <div className="bg-white/80 p-2 rounded-xl text-sky-700 w-fit shadow-sm"><product.icon size={18} /></div>
                                                    <div>
                                                        <p className="text-[8px] font-black uppercase text-sky-400 mb-0.5 leading-none">{product.category}</p>
                                                        <h3 className="font-extrabold text-slate-800 text-xs mb-1 leading-tight">{product.name}</h3>
                                                        <p className="text-sky-700 font-black text-lg">{formatMoney(product.price)}</p>
                                                    </div>
                                                </button>
                                            ))}
                                        </div>
                                    </div>
                                    
                                    <div className="w-80 lg:w-[380px] bg-white border-l flex flex-col shadow-2xl">
                                        <div className="p-5 border-b flex justify-between items-center bg-sky-50/20">
                                            <h2 className="font-black text-lg flex items-center gap-2 text-sky-900 uppercase tracking-tighter"><ShoppingCart size={18} /> Pedido</h2>
                                            <button onClick={() => setCart([])} className="text-[10px] font-black text-red-400 uppercase tracking-widest hover:text-red-600">Vaciar</button>
                                        </div>
                                        <div className="flex-1 overflow-y-auto p-4 space-y-3 custom-scrollbar">
                                            {cart.map(item => (
                                                <div key={item.id} className="bg-white p-3 rounded-xl border flex items-center gap-2 shadow-sm">
                                                    <div className="flex-1">
                                                        <p className="text-xs font-black text-slate-800 leading-tight mb-1">{item.name}</p>
                                                        <p className="text-[9px] text-sky-400 font-bold uppercase">{formatMoney(item.price)}</p>
                                                    </div>
                                                    <div className="flex items-center bg-slate-50 rounded-lg p-1 border">
                                                        <button onClick={() => updateQuantity(item.id, -1)} className="w-6 h-6 flex items-center justify-center text-slate-400 hover:text-sky-600"><Minus size={12}/></button>
                                                        <span className="w-6 text-center text-[10px] font-black text-slate-700">{item.quantity}</span>
                                                        <button onClick={() => updateQuantity(item.id, 1)} className="w-6 h-6 flex items-center justify-center text-slate-400 hover:text-sky-600"><Plus size={12}/></button>
                                                    </div>
                                                    <div className="text-right flex flex-col items-end w-20">
                                                        <p className="font-black text-sky-700 text-xs">{formatMoney(item.price * item.quantity)}</p>
                                                        <button onClick={() => removeFromCart(item.id)} className="text-red-300 hover:text-red-500 p-0.5"><Trash2 size={12} /></button>
                                                    </div>
                                                </div>
                                            ))}
                                            {cart.length === 0 && <div className="h-full flex flex-col items-center justify-center text-slate-200 opacity-50 py-20"><ShoppingCart size={40} className="mb-4" /><p className="font-black text-[10px] uppercase">Esperando Productos</p></div>}
                                        </div>
                                        <div className="p-6 bg-[#001d3d] text-white rounded-t-[2.5rem] shadow-inner">
                                            <div className="flex justify-between items-center mb-4 px-2">
                                                <span className="text-[10px] font-black text-sky-400 uppercase tracking-widest">Total Neto</span>
                                                <span className="text-3xl font-black text-white tracking-tighter">{formatMoney(total)}</span>
                                            </div>
                                            <button onClick={() => setIsCheckoutOpen(true)} disabled={cart.length === 0} className="w-full bg-sky-500 text-white py-4 rounded-[1.5rem] font-black text-sm uppercase tracking-widest shadow-xl active:scale-95 transition-all disabled:bg-slate-800">Procesar Cobro</button>
                                        </div>
                                    </div>
                                </div>
                            )}

                            {currentView === 'tours' && (
                                <div className="flex-1 p-8 overflow-y-auto bg-sky-50/20">
                                    <div className="max-w-4xl mx-auto">
                                        <h2 className="text-3xl font-black text-slate-900 tracking-tighter uppercase mb-8">Gestión de Recorridos</h2>
                                        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
                                            <div className="bg-white p-6 rounded-[2.5rem] border shadow-lg">
                                                <label className="block text-[9px] font-black text-slate-400 uppercase mb-3 tracking-widest text-center">Seleccionar Guía</label>
                                                <div className="space-y-2 mb-6">
                                                    {GUIDES.map(guide => (
                                                        <button key={guide} onClick={() => setSelectedGuide(guide)} className={`w-full p-3 rounded-xl text-left font-bold text-xs border-2 transition-all ${selectedGuide === guide ? 'bg-sky-50 border-sky-500 text-sky-700 shadow-sm' : 'bg-slate-50 border-transparent text-slate-400'}`}>{guide}</button>
                                                    ))}
                                                </div>
                                                <button onClick={handleCreateTour} disabled={!selectedGuide} className="w-full bg-emerald-600 text-white py-4 rounded-[1.5rem] font-black text-xs flex items-center justify-center gap-2 active:scale-95 shadow-md">
                                                    <MessageCircle size={18} /> Notificar por WSP
                                                </button>
                                            </div>
                                            <div className="lg:col-span-2 space-y-3">
                                                <h3 className="font-black uppercase text-[9px] tracking-widest text-slate-400 ml-2 mb-3">Programados (Margen 15 min)</h3>
                                                {tourHistory.map(tour => (
                                                    <div key={tour.id} className="bg-white p-4 rounded-3xl border shadow-sm flex items-center justify-between group hover:shadow-md transition-all">
                                                        <div className="flex items-center gap-4">
                                                            <div className="bg-sky-50 text-sky-600 p-3 rounded-2xl font-black text-lg shadow-inner">{tour.time}</div>
                                                            <div><p className="font-black text-slate-800 text-sm uppercase leading-none mb-1">{tour.guide}</p><p className="text-[9px] text-slate-400 font-bold uppercase tracking-widest">Creado a las {tour.created}</p></div>
                                                        </div>
                                                    </div>
                                                ))}
                                                {tourHistory.length === 0 && <div className="p-16 text-center text-slate-300 font-black uppercase text-[10px] border-2 border-dashed rounded-[2.5rem]">Sin recorridos hoy</div>}
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            )}

                            {currentView === 'history' && (
                                <div className="flex-1 p-8 overflow-y-auto bg-slate-50">
                                    <div className="max-w-3xl mx-auto space-y-3">
                                        <h2 className="text-3xl font-black text-slate-900 tracking-tighter uppercase mb-6 text-center">Bitácora de Ventas</h2>
                                        {salesHistory.map(sale => (
                                            <div key={sale.id} className="bg-white p-5 rounded-3xl border shadow-sm flex items-center justify-between hover:border-sky-200 transition-all">
                                                <div className="flex items-center gap-4">
                                                    <div className="w-10 h-10 bg-sky-100 text-sky-600 rounded-xl flex items-center justify-center font-black text-[9px] uppercase tracking-tighter shadow-inner">POS</div>
                                                    <div><p className="text-xs font-black text-slate-800 leading-none mb-1 uppercase">{sale.id}</p><p className="text-[9px] text-slate-400 font-bold uppercase tracking-widest">{sale.time} • {sale.method}</p></div>
                                                </div>
                                                <p className="text-xl font-black text-sky-800 tracking-tight">{formatMoney(sale.total)}</p>
                                            </div>
                                        ))}
                                        {salesHistory.length === 0 && <p className="text-center py-20 text-slate-300 font-black uppercase text-[10px]">Vacío</p>}
                                    </div>
                                </div>
                            )}

                            {currentView === 'reports' && (
                                <div className="flex-1 p-8 overflow-y-auto">
                                    <div className="max-w-4xl mx-auto">
                                        <h2 className="text-3xl font-black text-slate-900 tracking-tighter uppercase mb-10 text-center">Reporte de Inventario</h2>
                                        <div className="bg-white rounded-[2.5rem] border shadow-xl overflow-hidden">
                                            <table className="w-full text-left">
                                                <thead className="bg-slate-50 border-b">
                                                    <tr>
                                                        <th className="px-6 py-4 text-[9px] font-black text-slate-400 uppercase tracking-widest">Producto Marino</th>
                                                        <th className="px-6 py-4 text-[9px] font-black text-slate-400 uppercase tracking-widest text-center">Unidades</th>
                                                        <th className="px-6 py-4 text-[9px] font-black text-slate-400 uppercase tracking-widest text-right">Recaudado</th>
                                                    </tr>
                                                </thead>
                                                <tbody className="divide-y divide-slate-50">
                                                    {productSalesReport.map(p => (
                                                        <tr key={p.id} className={p.soldCount > 0 ? 'bg-white' : 'opacity-25'}>
                                                            <td className="px-6 py-4"><p className="font-black text-slate-800 text-xs uppercase tracking-tight">{p.name}</p></td>
                                                            <td className="px-6 py-4 text-center"><span className="text-lg font-black text-sky-700">{p.soldCount}</span></td>
                                                            <td className="px-6 py-4 text-right font-bold text-slate-500 text-xs">{formatMoney(p.totalRevenue)}</td>
                                                        </tr>
                                                    ))}
                                                </tbody>
                                            </table>
                                        </div>
                                    </div>
                                </div>
                            )}
                        </div>
                    </main>

                    {/* Checkout Modal */}
                    {isCheckoutOpen && (
                        <div className="fixed inset-0 bg-slate-900/90 backdrop-blur-xl z-[100] flex items-center justify-center p-4">
                            <div className="bg-white w-full max-w-sm rounded-[3.5rem] shadow-2xl overflow-hidden animate-in zoom-in duration-300">
                                <div className="p-8 text-center bg-[#001d3d] text-white">
                                    <p className="text-sky-400 text-[9px] font-black uppercase mb-3 tracking-widest">Total Neto Venta</p>
                                    <h3 className="text-5xl font-black tracking-tighter">{formatMoney(total)}</h3>
                                </div>
                                <div className="p-8 space-y-6 text-center">
                                    <div className="flex gap-3">
                                        <button onClick={() => setPaymentMethod('efectivo')} className={`flex-1 p-4 rounded-2xl border-2 transition-all text-[10px] font-black uppercase ${paymentMethod === 'efectivo' ? 'bg-sky-50 border-sky-500 text-sky-700 shadow-sm' : 'text-slate-300 border-transparent'}`}>Efectivo</button>
                                        <button onClick={() => setPaymentMethod('debito')} className={`flex-1 p-4 rounded-2xl border-2 transition-all text-[10px] font-black uppercase ${paymentMethod === 'debito' ? 'bg-blue-50 border-blue-500 text-blue-700 shadow-sm' : 'text-slate-300 border-transparent'}`}>Débito</button>
                                    </div>
                                    {paymentMethod === 'efectivo' && (
                                        <div className="space-y-4">
                                            <input type="number" className="w-full bg-slate-50 border-2 p-5 rounded-[2rem] text-4xl font-black text-center shadow-inner outline-none focus:border-sky-500" placeholder="Monto Recibido" value={cashReceived} onChange={(e) => setCashReceived(e.target.value)} autoFocus />
                                            <div className="p-5 rounded-2xl bg-emerald-50 text-center border border-emerald-100">
                                                <p className="text-emerald-700 font-black uppercase text-[8px] mb-1 tracking-widest">Vuelto</p>
                                                <p className="text-3xl font-black text-emerald-600">{formatMoney(Math.max(0, (parseFloat(cashReceived) || 0) - total))}</p>
                                            </div>
                                        </div>
                                    )}
                                    <button onClick={finalizeSale} disabled={paymentMethod === 'efectivo' && (!cashReceived || parseFloat(cashReceived) < total)} className="w-full bg-sky-600 text-white py-4 rounded-3xl font-black text-sm shadow-xl active:scale-95 transition-all uppercase tracking-widest">Confirmar</button>
                                    <button onClick={() => { setIsCheckoutOpen(false); setCashReceived(''); }} className="w-full text-slate-400 font-bold uppercase text-[9px] tracking-widest">Regresar</button>
                                </div>
                            </div>
                        </div>
                    )}
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>

