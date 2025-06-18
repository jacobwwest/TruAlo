// pages/index.tsx
import { useState, useEffect } from 'react';
import { createClient } from '@supabase/supabase-js';
import { loadStripe } from '@stripe/stripe-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);

const stripePromise = loadStripe('pk_test_xxx'); // Replace with actual public key

export default function Home() {
  const [user, setUser] = useState<any>(null);
  const [commodityUsage, setCommodityUsage] = useState(0);
  const [mileage, setMileage] = useState(0);
  const [buffer, setBuffer] = useState(0.1);
  const [truckCapacity, setTruckCapacity] = useState(1);
  const [truckTime, setTruckTime] = useState(0);
  const [totalTrucks, setTotalTrucks] = useState(0);

  useEffect(() => {
    supabase.auth.getUser().then(({ data: { user } }) => setUser(user));
  }, []);

  useEffect(() => {
    const baseTime = 1.42 * mileage;
    setTruckTime(baseTime);
    const adjustedUsage = commodityUsage * (1 + buffer);
    const trucks = adjustedUsage / (truckCapacity / baseTime);
    setTotalTrucks(Math.ceil(trucks));
  }, [commodityUsage, mileage, buffer, truckCapacity]);

  async function saveCalculation() {
    if (!user) return;
    await supabase.from('calculations').insert({
      user_id: user.id,
      commodity_usage: commodityUsage,
      mileage,
      truck_capacity: truckCapacity,
      total_trucks: totalTrucks,
    });
  }

  return (
    <main className="p-4">
      <h1 className="text-2xl font-bold mb-4">TruAlo Truck Calculator</h1>
      <input
        type="number"
        placeholder="Commodity Usage"
        value={commodityUsage}
        onChange={(e) => setCommodityUsage(parseFloat(e.target.value))}
        className="block border p-2 mb-2"
      />
      <input
        type="number"
        placeholder="Mileage"
        value={mileage}
        onChange={(e) => setMileage(parseFloat(e.target.value))}
        className="block border p-2 mb-2"
      />
      <input
        type="number"
        placeholder="Truck Capacity"
        value={truckCapacity}
        onChange={(e) => setTruckCapacity(parseFloat(e.target.value))}
        className="block border p-2 mb-2"
      />
      <p className="mb-2">One-way Travel Time: {truckTime.toFixed(2)} hrs</p>
      <p className="font-bold">Total Trucks Needed: {totalTrucks}</p>
      <button
        className="bg-blue-500 text-white px-4 py-2 mt-4"
        onClick={saveCalculation}
      >
        Save Calculation
      </button>
    </main>
  );
}