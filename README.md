# TruAlo
Truck Allocation Calculator 
// tru-alo/pages/index.tsx

import { useEffect, useState } from "react";
import { createClient } from "@supabase/supabase-js";

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);

export default function Home() {
  const [user, setUser] = useState(null);
  const [calculation, setCalculation] = useState({
    commodityUsage: 0,
    truckCapacity: 1,
    buffer: 0.1,
    travelDistance: 0,
  });
  const [result, setResult] = useState<number | null>(null);

  useEffect(() => {
    supabase.auth.getUser().then(({ data }) => {
      if (data?.user) {
        setUser(data.user);
      }
    });
  }, []);

  const handleLogin = async () => {
    const { error } = await supabase.auth.signInWithOtp({
      email: prompt("Enter your email") || "",
    });
    if (error) alert("Login error: " + error.message);
  };

  const calculateTrucks = () => {
    const usageWithBuffer = calculation.commodityUsage * (1 + calculation.buffer);
    const travelTime = 1.42 * calculation.travelDistance;
    const trucksNeeded = usageWithBuffer / (calculation.truckCapacity / travelTime);
    setResult(Math.ceil(trucksNeeded));
  };

  const handleSave = async () => {
    if (!user) return alert("Please log in first.");
    await supabase.from("calculations").insert({
      user_id: user.id,
      data: calculation,
      result,
    });
    alert("Calculation saved!");
  };

  return (
    <main className="p-6 max-w-xl mx-auto text-center">
      <h1 className="text-2xl font-bold mb-4">TruAlo - Truck Allocation Calculator</h1>

      {!user ? (
        <button onClick={handleLogin} className="bg-blue-600 text-white px-4 py-2 rounded">
          Log in via Email
        </button>
      ) : (
        <div className="space-y-4">
          <div>
            <label>Commodity Usage (tons): </label>
            <input
              type="number"
              value={calculation.commodityUsage}
              onChange={(e) =>
                setCalculation({ ...calculation, commodityUsage: Number(e.target.value) })
              }
              className="border p-2 rounded"
            />
          </div>

          <div>
            <label>Truck Capacity (tons): </label>
            <input
              type="number"
              value={calculation.truckCapacity}
              onChange={(e) =>
                setCalculation({ ...calculation, truckCapacity: Number(e.target.value) })
              }
              className="border p-2 rounded"
            />
          </div>

          <div>
            <label>Travel Distance (miles): </label>
            <input
              type="number"
              value={calculation.travelDistance}
              onChange={(e) =>
                setCalculation({ ...calculation, travelDistance: Number(e.target.value) })
              }
              className="border p-2 rounded"
            />
          </div>

          <button
            onClick={calculateTrucks}
            className="bg-green-600 text-white px-4 py-2 rounded"
          >
            Calculate Trucks
          </button>

          {result !== null && (
            <div className="text-lg font-semibold">Total Trucks Needed: {result}</div>
          )}

          <button
            onClick={handleSave}
            className="bg-gray-800 text-white px-4 py-2 rounded"
          >
            Save Calculation
          </button>
        </div>
      )}
    </main>
  );
}
